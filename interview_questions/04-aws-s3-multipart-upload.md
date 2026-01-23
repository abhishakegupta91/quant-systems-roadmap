# AWS S3 Multipart Upload

## Overview

Multipart upload is how you efficiently upload large files to S3 by splitting them into parts.

## What is Multipart Upload?

Instead of uploading one huge file in a single request, you split it into smaller parts and upload each independently. S3 then combines them into one object.

## Why use it?

- **Better throughput:** Upload multiple parts in parallel → faster upload.
- **Resilience:** If one part fails, retry just that part, not the entire file.
- **Mandatory for large files:** Recommended for files > 5GB.
- **Control:** Can adjust part size based on network conditions.

## How it works (3-step process)

### Step 1: Initiate Multipart Upload
```
Request: POST /bucket/key?uploads
Response: uploadId (unique identifier)
```

### Step 2: Upload Parts
```
For each part:
  Request: PUT /bucket/key?partNumber=1&uploadId=...
  Body: part data (5MB - 5GB per part)
  Response: ETag (checksum)
  
Keep track of partNumber -> ETag mapping
```

### Step 3: Complete Multipart Upload
```
Request: POST /bucket/key?uploadId=...
Body: XML list of (partNumber, ETag) pairs
Response: Final object created in S3
```

## Python Implementation Example

```python
import boto3
import concurrent.futures
from pathlib import Path

s3_client = boto3.client('s3')

def multipart_upload_file(bucket, key, file_path, part_size=5 * 1024 * 1024):
    """
    Upload large file using multipart upload.
    
    Args:
        bucket: S3 bucket name
        key: S3 object key (path)
        file_path: local file path
        part_size: size of each part (default 5MB)
    """
    
    file_size = Path(file_path).stat().st_size
    num_parts = (file_size + part_size - 1) // part_size
    
    # Step 1: Initiate multipart upload
    print(f"Initiating multipart upload for {file_path} ({num_parts} parts)")
    response = s3_client.create_multipart_upload(Bucket=bucket, Key=key)
    upload_id = response['UploadId']
    
    try:
        # Step 2: Upload parts (in parallel for speed)
        parts = []
        
        with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
            futures = []
            
            with open(file_path, 'rb') as f:
                for part_num in range(1, num_parts + 1):
                    start = (part_num - 1) * part_size
                    end = min(start + part_size, file_size)
                    
                    f.seek(start)
                    part_data = f.read(end - start)
                    
                    # Submit upload task to thread pool
                    future = executor.submit(
                        upload_part,
                        bucket, key, upload_id, part_num, part_data
                    )
                    futures.append((part_num, future))
            
            # Collect results
            for part_num, future in futures:
                etag = future.result()
                parts.append({'PartNumber': part_num, 'ETag': etag})
        
        # Sort by part number
        parts.sort(key=lambda x: x['PartNumber'])
        
        # Step 3: Complete multipart upload
        print(f"Completing multipart upload with {len(parts)} parts")
        response = s3_client.complete_multipart_upload(
            Bucket=bucket,
            Key=key,
            UploadId=upload_id,
            MultipartUpload={'Parts': parts}
        )
        
        print(f"Upload successful! Object: {response['Location']}")
        return response
        
    except Exception as e:
        print(f"Upload failed: {e}. Aborting multipart upload.")
        s3_client.abort_multipart_upload(
            Bucket=bucket,
            Key=key,
            UploadId=upload_id
        )
        raise

def upload_part(bucket, key, upload_id, part_num, part_data):
    """Upload a single part and return ETag"""
    print(f"Uploading part {part_num} ({len(part_data)} bytes)")
    response = s3_client.upload_part(
        Bucket=bucket,
        Key=key,
        PartNumber=part_num,
        UploadId=upload_id,
        Body=part_data
    )
    etag = response['ETag']
    print(f"Part {part_num} uploaded. ETag: {etag}")
    return etag

# Usage
if __name__ == '__main__':
    multipart_upload_file(
        bucket='my-bucket',
        key='videos/large-video.mp4',
        file_path='/local/path/video.mp4',
        part_size=10 * 1024 * 1024  # 10MB parts
    )
```

## Pre-signed URLs for Direct Client Uploads

### Generate Pre-signed Upload URL

```python
def get_multipart_upload_urls(bucket, key, part_count, part_size):
    """Generate pre-signed URLs for multipart upload"""
    s3_client = boto3.client('s3')
    
    # Initiate multipart upload
    response = s3_client.create_multipart_upload(Bucket=bucket, Key=key)
    upload_id = response['UploadId']
    
    # Generate pre-signed URLs for each part
    upload_urls = []
    for part_number in range(1, part_count + 1):
        presigned_url = s3_client.generate_presigned_url(
            'upload_part',
            Params={
                'Bucket': bucket,
                'Key': key,
                'PartNumber': part_number,
                'UploadId': upload_id
            },
            ExpiresIn=3600  # 1 hour
        )
        upload_urls.append({
            'partNumber': part_number,
            'url': presigned_url
        })
    
    return {
        'uploadId': upload_id,
        'urls': upload_urls
    }
```

### Client-side Upload (JavaScript/React)

```javascript
async function uploadLargeFile(file, uploadUrls) {
    const partSize = 5 * 1024 * 1024; // 5MB
    const totalParts = Math.ceil(file.size / partSize);
    
    const uploadPromises = [];
    
    for (let partNumber = 1; partNumber <= totalParts; partNumber++) {
        const start = (partNumber - 1) * partSize;
        const end = Math.min(start + partSize, file.size);
        const chunk = file.slice(start, end);
        
        const uploadPromise = fetch(uploadUrls[partNumber - 1].url, {
            method: 'PUT',
            body: chunk
        }).then(response => response.headers.get('ETag'));
        
        uploadPromises.push(uploadPromise);
    }
    
    // Wait for all parts to upload
    const etags = await Promise.all(uploadPromises);
    
    // Complete the upload
    const completeResponse = await fetch('/complete-upload', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            uploadId: uploadUrls.uploadId,
            parts: etags.map((etag, index) => ({
                PartNumber: index + 1,
                ETag: etag
            }))
        })
    });
    
    return completeResponse.json();
}
```

## Advanced Features

### Pause and Resume Upload

```python
class ResumableUploader:
    def __init__(self, bucket, key, file_path):
        self.bucket = bucket
        self.key = key
        self.file_path = file_path
        self.upload_state = self.load_upload_state()
    
    def load_upload_state(self):
        """Load upload progress from disk"""
        state_file = f"{self.file_path}.upload_state"
        if Path(state_file).exists():
            with open(state_file, 'r') as f:
                return json.load(f)
        return None
    
    def save_upload_state(self, state):
        """Save upload progress to disk"""
        state_file = f"{self.file_path}.upload_state"
        with open(state_file, 'w') as f:
            json.dump(state, f)
    
    def resume_upload(self):
        """Resume interrupted upload"""
        if not self.upload_state:
            return self.start_new_upload()
        
        # Check which parts are already uploaded
        uploaded_parts = self.upload_state['uploaded_parts']
        remaining_parts = self.get_remaining_parts(uploaded_parts)
        
        # Upload remaining parts
        for part_info in remaining_parts:
            self.upload_part(part_info)
        
        # Complete upload
        return self.complete_upload()
    
    def get_remaining_parts(self, uploaded_parts):
        """Calculate which parts still need to be uploaded"""
        part_size = 5 * 1024 * 1024
        file_size = Path(self.file_path).stat().st_size
        total_parts = (file_size + part_size - 1) // part_size
        
        uploaded_numbers = {p['partNumber'] for p in uploaded_parts}
        
        remaining = []
        for part_num in range(1, total_parts + 1):
            if part_num not in uploaded_numbers:
                remaining.append({
                    'partNumber': part_num,
                    'start': (part_num - 1) * part_size,
                    'end': min(part_num * part_size, file_size)
                })
        
        return remaining
```

### Progress Tracking

```python
class ProgressTracker:
    def __init__(self, total_parts):
        self.total_parts = total_parts
        self.completed_parts = 0
        self.start_time = time.time()
    
    def update_progress(self, part_number):
        self.completed_parts += 1
        progress = (self.completed_parts / self.total_parts) * 100
        elapsed = time.time() - self.start_time
        
        if self.completed_parts > 0:
            eta = (elapsed / self.completed_parts) * (self.total_parts - self.completed_parts)
            print(f"Part {part_number}/{self.total_parts} completed ({progress:.1f}%) - ETA: {eta:.1f}s")
        else:
            print(f"Part {part_number}/{self.total_parts} completed ({progress:.1f}%)")
```

### Error Handling and Retry Logic

```python
import time
from botocore.exceptions import ClientError

def upload_part_with_retry(bucket, key, upload_id, part_num, part_data, max_retries=3):
    """Upload part with exponential backoff retry"""
    for attempt in range(max_retries):
        try:
            response = s3_client.upload_part(
                Bucket=bucket,
                Key=key,
                PartNumber=part_num,
                UploadId=upload_id,
                Body=part_data
            )
            return response['ETag']
        
        except ClientError as e:
            if attempt == max_retries - 1:
                raise
            
            # Exponential backoff
            wait_time = (2 ** attempt) + 1
            print(f"Part {part_num} upload failed (attempt {attempt + 1}), retrying in {wait_time}s...")
            time.sleep(wait_time)
    
    raise Exception(f"Failed to upload part {part_num} after {max_retries} attempts")
```

## Interview Practice Questions

### AWS S3 & Large File Uploads

1. **Explain multipart upload and why it's useful.**
   - Split large file into parts (e.g., 5MB each).
   - Upload parts in parallel.
   - S3 assembles into single object.
   - Faster upload, resilient to failures.

2. **Implement a Python script to multipart upload a 1GB file.**
   - Initiate upload (CreateMultipartUpload).
   - Split file into parts.
   - Upload parts in parallel (ThreadPoolExecutor).
   - Collect ETags.
   - Complete upload (CompleteMultipartUpload).

3. **How would you implement "pause and resume" for uploads?**
   - Store uploadId in client.
   - Track which parts uploaded (in memory or DB).
   - On resume, upload remaining parts.
   - Specify part-specific range headers if resuming after network failure.

4. **What happens if one part fails during upload?**
   - Retry that specific part (not entire file).
   - Exponential backoff.
   - After max retries, abort upload (AbortMultipartUpload).

5. **How to securely allow direct client uploads to S3?**
   - Backend generates pre-signed URL (time-limited, limited scope).
   - Client uploads directly to pre-signed URL.
   - Backend never sees file content.
   - Pre-signed URL expires after 15 minutes.

6. **Design a flow to upload video directly to S3 and transcode.**
   - User gets pre-signed upload URL from backend.
   - User uploads directly to S3.
   - S3 triggers event (SNS/SQS).
   - Lambda or worker picks up event, transcodes.
   - Stores result back to S3.
   - Backend polls or uses webhook when ready.

7. **When would you choose multipart upload vs a simple `put_object` in S3?**
   - Multipart: Files > 100MB, need parallel uploads, want resume capability.
   - Simple: Small files (< 100MB), single request is sufficient.

8. **How would you implement direct-to-S3 uploads from a frontend app using pre-signed URLs?**
   - Backend generates pre-signed URLs for each part.
   - Frontend uploads parts directly to S3.
   - Frontend sends ETags to backend.
   - Backend completes multipart upload.

9. **What are the size limits for S3 multipart upload?**
   - Part size: 5MB to 5GB per part.
   - Maximum parts: 10,000 parts.
   - Maximum object size: 5TB.
   - Recommended: Use multipart for files > 100MB.

10. **How would you handle upload progress tracking for large files?**
    - Track completed parts vs total parts.
    - Calculate percentage complete.
    - Estimate remaining time based on upload speed.
    - Provide real-time feedback to user.

## Best Practices

### Part Size Selection
- **5-10MB parts:** Good balance for most scenarios.
- **Larger parts (50-100MB):** Fewer requests, more memory usage.
- **Smaller parts (1-5MB):** More parallelism, better for unreliable networks.

### Error Handling
- Always implement retry logic with exponential backoff.
- Use `AbortMultipartUpload` on failure to avoid storage costs.
- Validate file integrity with checksums.

### Security
- Use pre-signed URLs with short expiration times.
- Limit upload scope to specific bucket/key.
- Validate file types and sizes on the backend.
- Use IAM roles with least privilege.

### Performance
- Upload parts in parallel (4-8 concurrent uploads).
- Use appropriate part size for network conditions.
- Monitor upload metrics and optimize accordingly.

## Key Takeaways for Interviews

- **3-step process:** Initiate → Upload parts → Complete.
- **Benefits:** Parallel uploads, resilience, resume capability.
- **Implementation:** Use boto3, ThreadPoolExecutor for parallel uploads.
- **Security:** Pre-signed URLs, time-limited access.
- **Error handling:** Retry logic, cleanup on failure.
- **Use cases:** Large files, video uploads, backup systems.
