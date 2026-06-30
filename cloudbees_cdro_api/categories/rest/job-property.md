# REST API: Jobs and Properties

## Overview

Jobs are runtime instances created when a procedure or pipeline runs. Properties are key-value data attached to nearly any CloudBees CD/RO object.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List jobs | GET | `/jobs` |
| Get a job | GET | `/jobs/{jobId}` |
| Abort a job | PUT | `/jobs/{jobId}` |
| Delete a job | DELETE | `/jobs/{jobId}` |
| List job steps | GET | `/jobs/{jobId}/jobSteps` |
| Get a job step | GET | `/jobs/{jobId}/jobSteps/{stepName}` |
| Get job step log | GET | `/jobs/{jobId}/jobSteps/{stepName}/log` |
| Get a property | GET | `/properties` |
| Set a property | POST | `/properties` |
| Delete a property | DELETE | `/properties` |

---

## 1. Get a Job

**GET** `/jobs/{jobId}`

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/jobs/c3d4e5f6-7890-12ab-cdef-34567890abcd" \
  -H "sessionid: <API-token>"
```

### Python example

```python
job_id = "c3d4e5f6-7890-12ab-cdef-34567890abcd"
result = cdro.get(f"jobs/{job_id}")
job = result["job"]
print(f"Status: {job['status']}, Outcome: {job['outcome']}")
```

### Example JSON response

```json
{
  "job": {
    "jobId": "c3d4e5f6-7890-12ab-cdef-34567890abcd",
    "jobName": "Deploy App - 2024-06-30",
    "procedureName": "Deploy App",
    "projectName": "Online Banking",
    "status": "completed",
    "outcome": "success",
    "start": "2024-06-30T12:15:00.000Z",
    "finish": "2024-06-30T12:22:47.000Z",
    "elapsedTime": 467,
    "launchedBy": "admin"
  }
}
```

---

## 2. List Jobs

**GET** `/jobs`

### Optional query parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | string | Filter by project |
| `procedureName` | string | Filter by procedure name |
| `status` | string | Filter by status: `running`, `completed`, `waiting` |
| `outcome` | string | Filter by outcome: `success`, `error`, `warning` |
| `numObjects` | integer | Max number of jobs to return (default: 25) |
| `startIndex` | integer | Offset for pagination |

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/jobs?projectName=Online%20Banking&status=running" \
  -H "sessionid: <API-token>"
```

### Python example

```python
result = cdro.get(
    "jobs",
    projectName="Online Banking",
    status="running",
    numObjects=10
)
for job in result.get("job", []):
    print(job["jobId"], job["status"])
```

---

## 3. Abort a Job (PUT action)

**PUT** `/jobs/{jobId}`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `abortJob` |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `reason` | string | Human-readable explanation for the abort |
| `force` | boolean | Force abort even if agent is unresponsive |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/jobs/c3d4e5f6-7890-12ab-cdef-34567890abcd" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "abortJob",
    "reason": "Manual abort - wrong version deployed"
  }'
```

### Python example

```python
import requests

job_id = "c3d4e5f6-7890-12ab-cdef-34567890abcd"
url = f"{cdro.base}/jobs/{job_id}"
result = requests.put(
    url,
    headers=cdro.headers,
    json={"request": "abortJob", "reason": "Manual abort"},
    verify=False
).json()
print(result["job"]["status"])
```

---

## 4. Get Job Step Log

**GET** `/jobs/{jobId}/jobSteps/{stepName}/log`

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/jobs/c3d4e5f6-7890-12ab-cdef-34567890abcd/jobSteps/Run%20Tests/log" \
  -H "sessionid: <API-token>"
```

### Python example

```python
import urllib.parse

job_id = "c3d4e5f6-7890-12ab-cdef-34567890abcd"
step_name = "Run Tests"
result = cdro.get(f"jobs/{job_id}/jobSteps/{urllib.parse.quote(step_name)}/log")
print(result)
```

### Example response

The response body is plain text (not JSON):

```
+ cd /opt/app
+ ./run-tests.sh
Running test suite...
✓ Unit test 1 passed
✓ Unit test 2 passed
All 47 tests passed in 12.3s
```

---

## 5. Get a Property

**GET** `/properties`

Properties are addressed by their full path using the `propertyName` query parameter:

- Job property: `/jobs/<jobId>/<propertyName>`
- Project property: `/projects/<projectName>/<propertyName>`
- Server-level property: `/<propertyName>`

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/properties?propertyName=/jobs/c3d4e5f6-7890-12ab-cdef-34567890abcd/deployVersion" \
  -H "sessionid: <API-token>"
```

### Python example

```python
job_id = "c3d4e5f6-7890-12ab-cdef-34567890abcd"
result = cdro.get("properties", propertyName=f"/jobs/{job_id}/deployVersion")
prop = result["property"]
print(f"Value: {prop['value']}")
```

### Example JSON response

```json
{
  "property": {
    "propertyId": "e5f6a7b8-9012-34cd-ef01-567890123abc",
    "propertyName": "deployVersion",
    "value": "2.3.1",
    "expandable": "1",
    "lastModifiedBy": "admin",
    "modifyTime": "2024-06-30T12:16:00.000Z"
  }
}
```

---

## 6. Set a Property

**POST** `/properties`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `propertyName` | string | Full property path |
| `value` | string | Property value |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `expandable` | boolean | Whether to expand property references in the value |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/properties" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "propertyName": "/projects/Online Banking/appVersion",
    "value": "2.3.1",
    "description": "Current deployed version",
    "expandable": true
  }'
```

### Python example

```python
result = cdro.post(
    "properties",
    propertyName="/projects/Online Banking/appVersion",
    value="2.3.1",
    description="Current deployed version",
    expandable=True
)
print(result["property"]["propertyId"])
```

---

## Notes

- **Job IDs vs names:** Job endpoints use the UUID `jobId`, not the human-readable `jobName`. Capture the `jobId` from the response when starting a procedure or pipeline run.
- **Property paths:** Property names must be full hierarchical paths starting with `/`.
- **Log retrieval:** Logs are returned as plain text. For large logs, use `offset` and `limit` query params to page through content.
- **Polling pattern:** Poll `GET /jobs/{jobId}` on the `status` field (`running` → `completed`) to wait for a job to finish.
- **Swagger UI:** Full parameter documentation at `https://<cdro-server>/rest/doc/v1.0/`.