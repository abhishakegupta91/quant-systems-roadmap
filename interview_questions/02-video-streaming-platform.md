# Video Streaming Platform Design

## Overview

Design a "mini YouTube/Netflix" platform focusing on upload pipeline, storage, and delivery via CDN.

## Requirements (clarify these first)

### Functional Requirements
- Users can upload videos.
- Users can watch videos with playback controls (play, pause, seek).
- Basic metadata: title, description, thumbnail, duration.
- Multiple video resolutions (adaptive bitrate streaming).

### Non-Functional Requirements
- Scale to millions of concurrent users.
- Low playback latency (start streaming in <2 seconds).
- Cost-efficient storage (videos are large).
- 99.9% availability.
- Support for uploads up to several GB.

## High-level Architecture

```
[User Client]
      ↓
[API Gateway / Auth]
      ↓
[Backend Service] ← [Database (metadata)]
      ↓
[S3 / Object Storage]
      ↓
[Transcoding Queue (RabbitMQ/Kafka)]
      ↓
[Worker Service] (converts to multiple formats)
      ↓
[S3 Output / HLS Segments]
      ↓
[CDN (CloudFront/Akamai)]
      ↓
[User Client] (plays adaptive video)
```

## Key Components

### 1. Upload Pipeline

#### Direct-to-S3 Upload (recommended)
- Backend generates a **pre-signed S3 URL** valid for 15 minutes.
- Client uploads directly to S3 (bypasses backend, faster).
- S3 triggers an event notification when upload completes.
- Backend enqueues a transcoding job.

#### Backend Flow
```python
import boto3

def get_upload_url(video_id, user_id):
    s3_client = boto3.client('s3')
    
    # Generate pre-signed URL
    presigned_url = s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': 'videos-raw',
            'Key': f'uploads/{user_id}/{video_id}/video.mp4'
        },
        ExpiresIn=900  # 15 minutes
    )
    
    # Store video metadata in DB
    db.insert_video({
        'id': video_id,
        'user_id': user_id,
        'status': 'UPLOADING',
        'created_at': now()
    })
    
    return presigned_url
```

### 2. Transcoding Service

After upload completes, a worker service:
1. Reads raw video from S3.
2. Transcodes to multiple resolutions:
   - 240p (low quality, ~200 kbps)
   - 480p (medium quality, ~800 kbps)
   - 720p (high quality, ~2.5 mbps)
   - 1080p (HD, ~5 mbps)
3. Segments video into small chunks (~10 seconds each) for **HLS** (HTTP Live Streaming).
4. Generates playlist file (`.m3u8`) listing all segments.
5. Uploads segments and playlist to S3.

#### Why HLS/DASH?
- Client requests small chunks sequentially.
- If network slows, client switches to lower quality mid-stream.
- Resilient to network hiccups.

### 3. Playback Flow

```
User clicks play on video
    ↓
Backend returns HLS manifest URL (e.g., https://cdn.example.com/videos/123.m3u8)
    ↓
Client fetches manifest from CDN
    ↓
Manifest lists segments:
  - segment_1.ts (240p)
  - segment_2.ts (240p)
  - segment_3.ts (480p) [switched quality]
  ...
    ↓
Client fetches segments sequentially from CDN
    ↓
Video plays with adaptive bitrate based on network
```

### 4. Database Schema (simplified)

```sql
-- Videos table
CREATE TABLE videos (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36),
    title VARCHAR(255),
    description TEXT,
    duration INT,  -- in seconds
    status ENUM('UPLOADING', 'PROCESSING', 'READY', 'FAILED'),
    thumbnail_url VARCHAR(500),
    s3_key VARCHAR(500),  -- path in S3
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    INDEX (user_id),
    INDEX (status)
);

-- Video segments (for HLS)
CREATE TABLE video_segments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    video_id VARCHAR(36),
    resolution VARCHAR(10),  -- 240p, 480p, 720p, etc.
    segment_number INT,
    segment_duration FLOAT,
    s3_url VARCHAR(500),
    FOREIGN KEY (video_id) REFERENCES videos(id),
    INDEX (video_id)
);

-- Watch history
CREATE TABLE watch_history (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(36),
    video_id VARCHAR(36),
    watched_until INT,  -- seconds watched
    watched_at TIMESTAMP,
    INDEX (user_id, watched_at),
    FOREIGN KEY (video_id) REFERENCES videos(id)
);

-- Video views count (separate table for fast aggregation)
CREATE TABLE video_stats (
    video_id VARCHAR(36) PRIMARY KEY,
    view_count BIGINT DEFAULT 0,
    like_count BIGINT DEFAULT 0,
    FOREIGN KEY (video_id) REFERENCES videos(id)
);
```

## Scaling Considerations

### For sudden viral videos
- **CDN caching:** CDN caches popular segments near users globally.
- **Database reads:** Use read replicas for metadata queries.
- **Object storage:** S3 already scales automatically.

### Transcoding bottleneck
- Use auto-scaling worker pool: if queue grows, spin up more workers.
- Prioritize popular videos for faster transcoding.

### Storage optimization
- Delete raw upload after successful transcoding.
- Archive old/unpopular videos to cheaper storage (S3 Glacier).

## Security & Piracy Prevention

### How to prevent video piracy / unauthorized downloads
- Use signed URLs (pre-signed, time-limited).
- Only return segments to authenticated users.
- Use encryption (encrypt segments, key from server).
- DRM (Digital Rights Management) for high-value content.

### Secure upload flow
```python
def get_upload_url(video_id, user_id):
    # Verify user has upload permissions
    if not user_can_upload(user_id):
        raise UnauthorizedError()
    
    # Generate time-limited, scope-limited pre-signed URL
    presigned_url = s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': 'videos-raw',
            'Key': f'uploads/{user_id}/{video_id}/video.mp4',
            'ContentType': 'video/mp4'
        },
        ExpiresIn=900  # 15 minutes
    )
    
    return presigned_url
```

## Advanced Features

### Recommendation System
- Collect watch history.
- Use collaborative filtering (if users watched A and B, recommend similar videos).
- Use content-based filtering (if user likes "comedy", recommend other comedy).
- ML model: train on watch history, predict next video.

### Adaptive Bitrate Streaming Implementation
```python
def create_hls_manifest(video_id, resolutions):
    """Create HLS manifest with multiple bitrates"""
    manifest = "#EXTM3U\n"
    manifest += "#EXT-X-VERSION:3\n"
    
    for resolution in resolutions:
        bitrate = resolution['bitrate']
        width = resolution['width']
        height = resolution['height']
        playlist_url = f"{video_id}_{height}p.m3u8"
        
        manifest += f"#EXT-X-STREAM-INF:BANDWIDTH={bitrate},RESOLUTION={width}x{height}\n"
        manifest += f"{playlist_url}\n"
    
    return manifest
```

### Handling Simultaneous Uploads
- Direct-to-S3 upload (bypasses backend).
- S3 automatically scales.
- Use multipart upload for large files.
- Distribute uploads across multiple S3 regions.

## Interview Practice Questions

### System Design: Video Streaming

1. **Walk through end-to-end flow when user uploads a video to when another user can play it.**
   - User uploads to pre-signed S3 URL.
   - Backend enqueues transcode job.
   - Worker transcodes to multiple formats/resolutions.
   - Generates HLS segments and playlist.
   - Uploads to S3.
   - Updates DB status to READY.
   - Viewer requests video, gets HLS manifest, streams segments from CDN.

2. **How would you handle a viral video with sudden spike in viewers?**
   - CDN caches segments near users.
   - Database uses read replicas.
   - S3 automatically scales.
   - API gateway load balances across backend servers.

3. **Design the database schema for videos, users, and watch history. Discuss indexes.**
   - videos: id, user_id, title, status, s3_key.
   - Indexes on user_id, status.
   - watch_history: user_id, video_id, watched_until.
   - Composite index on (user_id, watched_at) for fast "recent videos" queries.

4. **How would you implement "adaptive bitrate streaming"?**
   - Transcode video into multiple bitrates (240p @ 200kbps, 480p @ 800kbps, etc.).
   - Each bitrate has separate segment files.
   - HLS playlist lists all variants.
   - Player monitors download speed; switches variant mid-stream.

5. **How to prevent video piracy / unauthorized downloads?**
   - Use signed URLs (pre-signed, time-limited).
   - Only return segments to authenticated users.
   - Use encryption (encrypt segments, key from server).
   - DRM (Digital Rights Management) for high-value content.

6. **How would you design a "recommendation" system recommending videos to users?**
   - Collect watch history.
   - Use collaborative filtering (if users watched A and B, recommend similar videos).
   - Use content-based filtering (if user likes "comedy", recommend other comedy).
   - ML model: train on watch history, predict next video.

7. **How to handle simultaneous uploads from millions of users?**
   - Direct-to-S3 upload (bypasses backend).
   - S3 automatically scales.
   - Use multipart upload for large files.
   - Distribute uploads across multiple S3 regions.

8. **Where would you use a CDN and what kind of content would you cache?**
   - Cache video segments (HLS chunks) near users.
   - Cache thumbnails and metadata.
   - Cache popular playlists.
   - Don't cache user-specific content or admin endpoints.

9. **How would you track and store watch history efficiently for millions of users?**
   - Use separate watch_history table with composite index (user_id, watched_at).
   - Batch updates to reduce DB load.
   - Consider time-series database for analytics.
   - Archive old history to cold storage.

10. **How would you design the upload pipeline to handle large files (GBs) reliably?**
    - Use multipart upload for large files.
    - Implement progress tracking.
    - Handle pause/resume functionality.
    - Use checksums to verify integrity.
    - Queue processing to avoid blocking uploads.

## Key Takeaways for Interviews

- **Upload flow:** Pre-signed URLs → Direct S3 upload → Event trigger → Transcoding queue.
- **Streaming:** HLS/DASH for adaptive bitrate, CDN for low latency.
- **Scaling:** CDN for content, read replicas for metadata, auto-scaling workers.
- **Storage:** S3 for raw and processed videos, lifecycle policies for cost optimization.
- **Security:** Pre-signed URLs, encryption, authentication.
- **Performance:** Segment caching, database indexing, parallel processing.
