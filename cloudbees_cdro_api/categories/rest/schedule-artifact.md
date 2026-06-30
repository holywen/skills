# REST API: Schedules and Artifacts

## Overview

Schedules trigger procedures or pipelines automatically on a time-based or event-based basis (cron, interval, or continuous integration). Artifacts are versioned binary packages stored in the CloudBees CD/RO artifact repository and retrieved during deployments. This file covers creating and running schedules, and publishing/retrieving artifact versions.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List schedules | GET | `/projects/{projectName}/schedules` |
| Get a schedule | GET | `/projects/{projectName}/schedules/{scheduleName}` |
| Create a schedule | POST | `/projects/{projectName}/schedules` |
| Update a schedule | PUT | `/projects/{projectName}/schedules/{scheduleName}` |
| Delete a schedule | DELETE | `/projects/{projectName}/schedules/{scheduleName}` |
| Run a schedule | PUT | `/projects/{projectName}/schedules/{scheduleName}` |
| List artifacts | GET | `/artifacts` |
| Get an artifact | GET | `/artifacts/{groupId}:{artifactKey}` |
| Create an artifact | POST | `/artifacts` |
| Delete an artifact | DELETE | `/artifacts/{groupId}:{artifactKey}` |
| List artifact versions | GET | `/artifacts/{groupId}:{artifactKey}/versions` |
| Get an artifact version | GET | `/artifacts/{groupId}:{artifactKey}/versions/{version}` |
| Publish artifact version | POST | `/artifacts/{groupId}:{artifactKey}/versions` |
| Retrieve artifact version | PUT | `/artifacts/{groupId}:{artifactKey}/versions/{version}` |
| Delete artifact version | DELETE | `/artifacts/{groupId}:{artifactKey}/versions/{version}` |

> **Artifact addressing:** Artifacts are addressed as `{groupId}:{artifactKey}` (e.g., `com.example:banking-app`). In URLs, the colon is literal: `/artifacts/com.example:banking-app`.
> **URI encoding:** Spaces in schedule names must be percent-encoded.

---

## 1. Create a Schedule

**POST** `/projects/{projectName}/schedules`

Schedules can trigger procedures or pipelines. The `scheduleDisabled` flag lets you create a schedule in paused state.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `scheduleName` | string | Name of the schedule |
| `procedureName` | string | Procedure to run (or `pipelineName` for pipelines) |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `pipelineName` | string | Pipeline to run (use instead of `procedureName`) |
| `description` | string | Human-readable description |
| `scheduleDisabled` | boolean | Create in disabled state (default: `false`) |
| `startTime` | string | Cron-style start time: `"HH:MM"` |
| `stopTime` | string | Cron-style stop time: `"HH:MM"` |
| `weekDays` | string | Comma-separated days: `"Mon,Wed,Fri"` or `"all"` |
| `months` | string | Comma-separated months: `"Jan,Feb"` or `"all"` |
| `monthDays` | string | Comma-separated month days: `"1,15"` or `"all"` |
| `interval` | string | Interval in minutes between runs |
| `priority` | string | Job priority: `low`, `normal`, `high`, `highest` |
| `actualParameter` | array | Fixed parameters: `[{actualParameterName, value}]` |
| `timeZone` | string | Timezone: `"America/New_York"` |
| `misfirePolicy` | string | `"runOnce"` or `"ignore"` |

### curl example — cron-style schedule

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/schedules" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "scheduleName": "Nightly Deploy",
    "procedureName": "Deploy App",
    "description": "Deploy to staging every weekday night",
    "startTime": "22:00",
    "stopTime": "23:00",
    "weekDays": "Mon,Tue,Wed,Thu,Fri",
    "months": "all",
    "monthDays": "all",
    "timeZone": "America/New_York",
    "actualParameter": [
      {"actualParameterName": "environment", "value": "staging"}
    ]
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/schedules",
    scheduleName="Nightly Deploy",
    procedureName="Deploy App",
    description="Deploy to staging every weekday night",
    startTime="22:00",
    weekDays="Mon,Tue,Wed,Thu,Fri",
    months="all",
    monthDays="all",
    timeZone="America/New_York"
)
print(result["schedule"]["scheduleId"])
```

### Example JSON response

```json
{
  "schedule": {
    "scheduleId": "a1b2c3d4-aaaa-1111-2222-333344445555",
    "scheduleName": "Nightly Deploy",
    "procedureName": "Deploy App",
    "projectName": "Online Banking",
    "startTime": "22:00",
    "weekDays": "Mon,Tue,Wed,Thu,Fri",
    "timeZone": "America/New_York",
    "scheduleDisabled": "0",
    "createTime": "2024-06-30T12:00:00.000Z",
    "nextRun": "2024-07-01T02:00:00.000Z"
  }
}
```

---

## 2. Run a Schedule on Demand (PUT action)

**PUT** `/projects/{projectName}/schedules/{scheduleName}`

Triggers a scheduled run immediately outside of the normal schedule using `request=runSchedule`.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runSchedule` |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/schedules/Nightly%20Deploy" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{"request": "runSchedule"}'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
schedule = "Nightly Deploy"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/schedules/{urllib.parse.quote(schedule)}"

result = requests.put(url, headers=cdro.headers,
                      json={"request": "runSchedule"}, verify=False).json()
print(result["job"]["jobId"])
```

### Example JSON response

```json
{
  "job": {
    "jobId": "b2c3d4e5-bbbb-2222-3333-444455556666",
    "jobName": "Deploy App - 2024-06-30 (Nightly Deploy)",
    "status": "running",
    "outcome": "none",
    "projectName": "Online Banking",
    "start": "2024-06-30T15:00:00.000Z"
  }
}
```

---

## 3. Publish an Artifact Version

**POST** `/artifacts/{groupId}:{artifactKey}/versions`

Publishes a new version of an artifact. The artifact must exist first (create it with `POST /artifacts` if needed).

> **Note:** File upload for artifact publishing is typically done via the `ectool` CLI (`publishArtifactVersion`) or the artifact uploader tool. The REST API endpoint for publishing accepts metadata; actual file content is sent as a multipart form upload.

### First: Create the Artifact (if it doesn't exist)

**POST** `/artifacts`

| Parameter | Type | Description |
|---|---|---|
| `groupId` | string | Maven-style group ID (e.g., `com.example`) |
| `artifactKey` | string | Artifact name/key (e.g., `banking-app`) |
| `description` | string | Artifact description (optional) |

### curl example — create artifact

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/artifacts" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "groupId": "com.example",
    "artifactKey": "banking-app",
    "description": "Banking application WAR artifact"
  }'
```

### curl example — publish version

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/artifacts/com.example:banking-app/versions" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "version": "2.3.1",
    "description": "Release 2.3.1 build",
    "repositoryName": "default"
  }'
```

### Python example

```python
import requests

# Create artifact if needed
cdro.post(
    "artifacts",
    groupId="com.example",
    artifactKey="banking-app",
    description="Banking application WAR artifact"
)

# Publish version
result = requests.post(
    f"{cdro.base}/artifacts/com.example:banking-app/versions",
    headers=cdro.headers,
    json={
        "version": "2.3.1",
        "description": "Release 2.3.1 build",
        "repositoryName": "default"
    },
    verify=False
).json()
print(result["artifactVersion"]["artifactVersionId"])
```

### Example JSON response

```json
{
  "artifactVersion": {
    "artifactVersionId": "c3d4e5f6-cccc-3333-4444-555566667777",
    "artifactKey": "banking-app",
    "groupId": "com.example",
    "version": "2.3.1",
    "repositoryName": "default",
    "description": "Release 2.3.1 build",
    "createTime": "2024-06-30T13:00:00.000Z",
    "size": 0
  }
}
```

---

## 4. Retrieve Artifact Versions

**GET** `/artifacts/{groupId}:{artifactKey}/versions`

Lists all available versions of an artifact.

### Optional query parameters

| Parameter | Type | Description |
|---|---|---|
| `numObjects` | integer | Max versions to return |
| `startIndex` | integer | Pagination offset |
| `version` | string | Filter by specific version string |

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/artifacts/com.example:banking-app/versions?numObjects=10" \
  -H "sessionid: <API-token>"
```

### Python example

```python
result = cdro.get(
    "artifacts/com.example:banking-app/versions",
    numObjects=10
)
for av in result.get("artifactVersion", []):
    print(av["version"], av["createTime"], av.get("size", "unknown"))
```

### Example JSON response

```json
{
  "artifactVersion": [
    {
      "artifactVersionId": "c3d4e5f6-cccc-3333-4444-555566667777",
      "groupId": "com.example",
      "artifactKey": "banking-app",
      "version": "2.3.1",
      "repositoryName": "default",
      "createTime": "2024-06-30T13:00:00.000Z",
      "size": 45678901
    },
    {
      "artifactVersionId": "d4e5f6a7-dddd-4444-5555-666677778888",
      "groupId": "com.example",
      "artifactKey": "banking-app",
      "version": "2.3.0",
      "repositoryName": "default",
      "createTime": "2024-06-20T11:00:00.000Z",
      "size": 44532100
    }
  ]
}
```

---

## Notes

- **Cron vs interval:** Schedules support either cron-style (day/time combinations) or interval-based (`interval` in minutes) triggering. These are mutually exclusive — use one or the other.
- **Artifact groupId:key format:** The colon separator is literal in the URL. Do not percent-encode it: `/artifacts/com.example:banking-app/versions` is correct.
- **Artifact file upload:** The actual binary upload is done via `ectool publishArtifactVersion` or the artifact HTTP upload endpoint (separate from the REST v1.0 API). The REST API manages metadata and version records.
- **Version retention:** Old artifact versions can be deleted with `DELETE /artifacts/{groupId}:{artifactKey}/versions/{version}` to manage storage.
- **Schedule misfires:** If the server was down when a schedule was supposed to fire, `misfirePolicy` controls behavior: `runOnce` fires once on restart, `ignore` skips the missed run.
- **Swagger UI:** Full parameter reference including schedule frequency options at `https://<cdro-server>/rest/doc/v1.0/`.