# JIRA-like Service Implementation

## Overview

Design and implement a JIRA-like ticketing system with core entities, REST APIs, and clean structure in Python using Test-Driven Development (TDD).

## Core Entities

### Data Models

```python
from datetime import datetime
from enum import Enum

class IssueStatus(Enum):
    TO_DO = "TO_DO"
    IN_PROGRESS = "IN_PROGRESS"
    DONE = "DONE"

class IssueType(Enum):
    BUG = "BUG"
    TASK = "TASK"
    STORY = "STORY"

class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4

class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email

class Project:
    def __init__(self, id, name, description):
        self.id = id
        self.name = name
        self.description = description

class Issue:
    def __init__(self, id, title, description, project_id, issue_type, priority, assignee_id=None):
        self.id = id
        self.title = title
        self.description = description
        self.project_id = project_id
        self.status = IssueStatus.TO_DO
        self.issue_type = issue_type
        self.priority = priority
        self.assignee_id = assignee_id
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
        self.comments = []
```

## Database Schema

```sql
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP
);

CREATE TABLE projects (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    created_by VARCHAR(36),
    created_at TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE TABLE issues (
    id VARCHAR(36) PRIMARY KEY,
    project_id VARCHAR(36),
    title VARCHAR(500),
    description TEXT,
    status ENUM('TO_DO', 'IN_PROGRESS', 'DONE') DEFAULT 'TO_DO',
    issue_type ENUM('BUG', 'TASK', 'STORY'),
    priority INT DEFAULT 2,
    assignee_id VARCHAR(36),
    created_by VARCHAR(36),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    version INT DEFAULT 1,  -- for optimistic locking
    FOREIGN KEY (project_id) REFERENCES projects(id),
    FOREIGN KEY (assignee_id) REFERENCES users(id),
    FOREIGN KEY (created_by) REFERENCES users(id),
    INDEX (project_id, status),
    INDEX (assignee_id)
);

CREATE TABLE issue_comments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    issue_id VARCHAR(36),
    author_id VARCHAR(36),
    content TEXT,
    created_at TIMESTAMP,
    FOREIGN KEY (issue_id) REFERENCES issues(id),
    FOREIGN KEY (author_id) REFERENCES users(id)
);

CREATE TABLE issue_audit (
    id INT AUTO_INCREMENT PRIMARY KEY,
    issue_id VARCHAR(36),
    changed_by VARCHAR(36),
    field_name VARCHAR(100),
    old_value VARCHAR(500),
    new_value VARCHAR(500),
    changed_at TIMESTAMP,
    FOREIGN KEY (issue_id) REFERENCES issues(id),
    FOREIGN KEY (changed_by) REFERENCES users(id),
    INDEX (issue_id, changed_at)
);
```

## REST API Endpoints

### FastAPI Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional, List
import uuid

app = FastAPI()

# ===== CREATE PROJECT =====
class CreateProjectRequest(BaseModel):
    name: str
    description: str

@app.post("/projects")
async def create_project(request: CreateProjectRequest, user_id: str):
    project_id = str(uuid.uuid4())
    db.insert_project({
        'id': project_id,
        'name': request.name,
        'description': request.description,
        'created_by': user_id
    })
    return {'id': project_id, 'name': request.name}

# ===== CREATE ISSUE =====
class CreateIssueRequest(BaseModel):
    title: str
    description: str
    issue_type: str  # BUG, TASK, STORY
    priority: int
    assignee_id: Optional[str] = None

@app.post("/projects/{project_id}/issues")
async def create_issue(project_id: str, request: CreateIssueRequest, user_id: str):
    issue_id = str(uuid.uuid4())
    db.insert_issue({
        'id': issue_id,
        'project_id': project_id,
        'title': request.title,
        'description': request.description,
        'status': 'TO_DO',
        'issue_type': request.issue_type,
        'priority': request.priority,
        'assignee_id': request.assignee_id,
        'created_by': user_id,
        'version': 1
    })
    return {'id': issue_id}

# ===== GET ISSUE =====
@app.get("/issues/{issue_id}")
async def get_issue(issue_id: str):
    issue = db.find_issue_by_id(issue_id)
    if not issue:
        raise HTTPException(status_code=404, detail="Issue not found")
    return issue

# ===== UPDATE ISSUE (optimistic locking) =====
class UpdateIssueRequest(BaseModel):
    status: Optional[str] = None
    assignee_id: Optional[str] = None
    version: int  # current version for locking

@app.patch("/issues/{issue_id}")
async def update_issue(issue_id: str, request: UpdateIssueRequest, user_id: str):
    issue = db.find_issue_by_id(issue_id)
    
    # Optimistic locking: check version
    if issue['version'] != request.version:
        raise HTTPException(status_code=409, detail="Issue was modified by another user")
    
    # Track audit
    if request.status and request.status != issue['status']:
        db.insert_audit({
            'issue_id': issue_id,
            'changed_by': user_id,
            'field_name': 'status',
            'old_value': issue['status'],
            'new_value': request.status
        })
    
    # Update issue
    db.update_issue(issue_id, {
        'status': request.status or issue['status'],
        'assignee_id': request.assignee_id or issue['assignee_id'],
        'version': issue['version'] + 1,
        'updated_at': datetime.now()
    })
    
    return {'id': issue_id, 'version': issue['version'] + 1}

# ===== SEARCH/FILTER ISSUES =====
@app.get("/projects/{project_id}/issues")
async def list_issues(
    project_id: str,
    status: Optional[str] = None,
    assignee_id: Optional[str] = None,
    priority: Optional[int] = None,
    skip: int = 0,
    limit: int = 20
):
    query = {'project_id': project_id}
    if status:
        query['status'] = status
    if assignee_id:
        query['assignee_id'] = assignee_id
    if priority:
        query['priority'] = priority
    
    issues = db.find_issues(query, skip=skip, limit=limit)
    return issues

# ===== ADD COMMENT =====
class AddCommentRequest(BaseModel):
    content: str

@app.post("/issues/{issue_id}/comments")
async def add_comment(issue_id: str, request: AddCommentRequest, user_id: str):
    db.insert_comment({
        'issue_id': issue_id,
        'author_id': user_id,
        'content': request.content
    })
    return {'status': 'ok'}
```

## Test-Driven Development Example

### Unit Tests with pytest

```python
import pytest
from fastapi.testclient import TestClient

client = TestClient(app)

def test_create_issue():
    """Test creating a new issue"""
    response = client.post(
        "/projects/proj-123/issues",
        json={
            'title': 'Login button broken',
            'description': 'Users cannot click login',
            'issue_type': 'BUG',
            'priority': 3
        },
        headers={'X-User-Id': 'user-1'}
    )
    assert response.status_code == 200
    data = response.json()
    assert 'id' in data

def test_update_issue_status():
    """Test changing issue status"""
    # First create an issue
    create_response = client.post("/projects/proj-123/issues", ...)
    issue_id = create_response.json()['id']
    
    # Get current version
    get_response = client.get(f"/issues/{issue_id}")
    version = get_response.json()['version']
    
    # Update status
    update_response = client.patch(
        f"/issues/{issue_id}",
        json={
            'status': 'IN_PROGRESS',
            'version': version
        },
        headers={'X-User-Id': 'user-1'}
    )
    assert update_response.status_code == 200

def test_concurrent_update_fails():
    """Test that concurrent updates fail with optimistic locking"""
    # User 1 fetches issue, version = 1
    response1 = client.get(f"/issues/issue-123")
    version1 = response1.json()['version']
    
    # User 2 fetches issue, version = 1
    response2 = client.get(f"/issues/issue-123")
    version2 = response2.json()['version']
    
    # User 2 updates first, increments version to 2
    client.patch(
        f"/issues/issue-123",
        json={'status': 'DONE', 'version': version2},
        headers={'X-User-Id': 'user-2'}
    )
    
    # User 1 tries to update with stale version, should fail
    update_response = client.patch(
        f"/issues/issue-123",
        json={'status': 'IN_PROGRESS', 'version': version1},
        headers={'X-User-Id': 'user-1'}
    )
    assert update_response.status_code == 409  # Conflict

def test_filter_issues_by_status():
    """Test filtering issues by status"""
    response = client.get("/projects/proj-123/issues?status=IN_PROGRESS")
    assert response.status_code == 200
    issues = response.json()
    for issue in issues:
        assert issue['status'] == 'IN_PROGRESS'
```

## Key Design Patterns

### Optimistic Locking
- Add a `version` column to issues.
- When updating, check that version matches; if not, someone else modified it.
- Increment version on each successful update.
- Prevents lost updates in concurrent scenarios.

### Audit Trail
- Track every change in `issue_audit` table.
- Useful for understanding who did what and when.
- Can implement "undo" functionality.

### Efficient Filtering
- Index by `project_id`, `status`, `assignee_id`.
- Composite index on (project_id, status) for common queries.
- Pagination with `skip` and `limit`.

## Service Layer Architecture

### Repository Pattern

```python
class IssueRepository:
    def __init__(self, db_connection):
        self.db = db_connection
    
    def find_by_id(self, issue_id: str) -> Optional[Issue]:
        # Database query to find issue
        pass
    
    def create(self, issue: Issue) -> Issue:
        # Insert new issue
        pass
    
    def update(self, issue: Issue) -> Issue:
        # Update existing issue
        pass
    
    def find_by_project(self, project_id: str, filters: dict) -> List[Issue]:
        # Find issues with filters
        pass

class IssueService:
    def __init__(self, repository: IssueRepository, audit_service: AuditService):
        self.repository = repository
        self.audit_service = audit_service
    
    def create_issue(self, project_id: str, title: str, description: str, 
                     issue_type: str, priority: int, assignee_id: str = None) -> Issue:
        # Business logic for creating issue
        issue = Issue(
            id=str(uuid.uuid4()),
            title=title,
            description=description,
            project_id=project_id,
            issue_type=issue_type,
            priority=priority,
            assignee_id=assignee_id
        )
        
        return self.repository.create(issue)
    
    def update_status(self, issue_id: str, new_status: str, user_id: str) -> Issue:
        issue = self.repository.find_by_id(issue_id)
        if not issue:
            raise IssueNotFoundError()
        
        old_status = issue.status
        issue.status = new_status
        issue.updated_at = datetime.now()
        
        # Track audit
        self.audit_service.track_change(
            issue_id=issue_id,
            field_name='status',
            old_value=old_status,
            new_value=new_status,
            changed_by=user_id
        )
        
        return self.repository.update(issue)
```

## Advanced Features

### Issue Dependencies

```sql
CREATE TABLE issue_dependencies (
    blocking_issue_id VARCHAR(36),
    blocked_issue_id VARCHAR(36),
    PRIMARY KEY (blocking_issue_id, blocked_issue_id),
    FOREIGN KEY (blocking_issue_id) REFERENCES issues(id),
    FOREIGN KEY (blocked_issue_id) REFERENCES issues(id)
);
```

### Full-Text Search

```python
# Using Elasticsearch
from elasticsearch import Elasticsearch

class IssueSearchService:
    def __init__(self, es_client):
        self.es = es_client
    
    def index_issue(self, issue: Issue):
        self.es.index(
            index='issues',
            id=issue.id,
            body={
                'title': issue.title,
                'description': issue.description,
                'project_id': issue.project_id,
                'status': issue.status,
                'assignee_id': issue.assignee_id
            }
        )
    
    def search(self, query: str, filters: dict = None) -> List[Issue]:
        search_body = {
            'query': {
                'bool': {
                    'must': [
                        {'multi_match': {'query': query, 'fields': ['title', 'description']}}
                    ]
                }
            }
        }
        
        if filters:
            for field, value in filters.items():
                search_body['query']['bool']['filter'].append({'term': {field: value}})
        
        response = self.es.search(index='issues', body=search_body)
        return [hit['_source'] for hit in response['hits']['hits']]
```

### Webhooks for Issue Updates

```python
@app.post("/webhooks/issue-updated")
async def issue_updated_webhook(payload: dict):
    issue_id = payload['issue_id']
    changes = payload['changes']
    
    # Notify subscribers
    for subscriber in get_subscribers(issue_id):
        send_notification(
            user=subscriber,
            message=f"Issue {issue_id} updated: {changes}",
            type='issue_updated'
        )
    
    return {'status': 'ok'}
```

## Interview Practice Questions

### JIRA-like Service & TDD

1. **Design the DB schema for a JIRA-like system.**
   - users, projects, issues, comments, audit.
   - Key: issue.status, issue.assignee, timestamps.
   - Composite indexes on (project_id, status).

2. **How would you implement optimistic locking to prevent lost updates?**
   - Add version column to issues.
   - Before update, check version matches.
   - If mismatch, return 409 Conflict.
   - Increment version on successful update.

3. **Describe the TDD approach: write tests first, then code.**
   - Write test: test_create_issue().
   - Test should fail initially (no implementation).
   - Write minimal code to pass test.
   - Refactor code for clarity.
   - Repeat for each feature.

4. **How to design API for filtering issues (status, assignee, priority)?**
   - GET /projects/{id}/issues?status=IN_PROGRESS&assignee=user123.
   - Build query incrementally based on filters.
   - Use indexes to optimize.
   - Support pagination (skip, limit).

5. **Implement an audit trail tracking who changed what and when.**
   - On every update, insert row in issue_audit.
   - Record field_name, old_value, new_value, changed_by, changed_at.
   - Use for compliance, debugging, "undo" feature.

6. **How to implement full-text search for issue descriptions?**
   - Add FULLTEXT index on issue.description.
   - Use MySQL MATCH AGAINST(...) or Elasticsearch.
   - Support phrase search, Boolean operators.

7. **How would you handle "issue dependencies" (Issue A blocks Issue B)?**
   - Add issue_dependencies table: (blocking_issue_id, blocked_issue_id).
   - When blocking issue moves to DONE, notify blocked issue assignee.
   - Prevent closing blocked issue if blocker not done.

8. **Explain how you would structure a Python service layer and repository layer for issues.**
   - Repository: Database operations (CRUD).
   - Service: Business logic, validation, orchestration.
   - Controller: HTTP endpoints, request/response handling.
   - Models: Data structures, validation.

9. **How would you implement search/filter for issues (by status, assignee, labels, full text)?**
   - Database queries for structured filters (status, assignee).
   - Elasticsearch for full-text search.
   - Combine both for comprehensive search.
   - Cache frequent queries.

10. **How would you design configuration and environment management for a service deployed to multiple environments?**
    - Environment variables for config.
    - Separate config files per environment.
    - Use dependency injection for config.
    - Secrets management for sensitive data.

## Key Takeaways for Interviews

- **Database design:** Proper relationships, indexes for performance.
- **API design:** RESTful principles, filtering, pagination.
- **Concurrency control:** Optimistic locking with version numbers.
- **TDD approach:** Write tests first, then implementation.
- **Audit trail:** Track all changes for compliance and debugging.
- **Service architecture:** Separate layers (repository, service, controller).
- **Performance:** Indexing, caching, pagination for scale.
