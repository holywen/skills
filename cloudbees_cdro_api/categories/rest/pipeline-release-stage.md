# REST API: Pipelines, Releases, and Stages

## Overview

Pipelines model multi-stage delivery workflows visually. Releases orchestrate multiple pipelines and deployment activities across time.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List pipelines | GET | `/projects/{projectName}/pipelines` |
| Get a pipeline | GET | `/projects/{projectName}/pipelines/{pipelineName}` |
| Create a pipeline | POST | `/projects/{projectName}/pipelines` |
| Run a pipeline | PUT | `/projects/{projectName}/pipelines/{pipelineName}` |
| List stages | GET | `/projects/{projectName}/pipelines/{pipelineName}/stages` |
| Create a stage | POST | `/projects/{projectName}/pipelines/{pipelineName}/stages` |
| Create a task | POST | `/projects/{projectName}/pipelines/{pipelineName}/stages/{stageName}/tasks` |
| List flow runtimes | GET | `/flowRuntimes` |
| Get a flow runtime | GET | `/flowRuntimes/{flowRuntimeId}` |
| Abort a flow runtime | PUT | `/flowRuntimes/{flowRuntimeId}` |
| List releases | GET | `/projects/{projectName}/releases` |
| Create a release | POST | `/projects/{projectName}/releases` |
| Start a release | PUT | `/projects/{projectName}/releases/{releaseName}` |

---

## 1. Create a Pipeline

**POST** `/projects/{projectName}/pipelines`

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/pipelines" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "pipelineName": "Continuous Delivery",
    "description": "Main CD pipeline for banking app"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/pipelines",
    pipelineName="Continuous Delivery",
    description="Main CD pipeline for banking app"
)
print(result["pipeline"]["pipelineId"])
```

---

## 2. Run a Pipeline (PUT action)

**PUT** `/projects/{projectName}/pipelines/{pipelineName}`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runPipeline` |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `actualParameter` | array | Runtime parameters as `[{actualParameterName, value}]` |
| `startingStage` | string | Name of the stage to start from |
| `pipelineRunName` | string | Custom label for this run |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/pipelines/Continuous%20Delivery" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "runPipeline",
    "pipelineRunName": "release-2.3.1",
    "actualParameter": [
      {"actualParameterName": "gitBranch", "value": "main"},
      {"actualParameterName": "version", "value": "2.3.1"}
    ]
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
pipeline = "Continuous Delivery"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/pipelines/{urllib.parse.quote(pipeline)}"

result = requests.put(url, headers=cdro.headers, json={
    "request": "runPipeline",
    "pipelineRunName": "release-2.3.1",
    "actualParameter": [
        {"actualParameterName": "gitBranch", "value": "main"},
        {"actualParameterName": "version", "value": "2.3.1"}
    ]
}, verify=False).json()

runtime_id = result["flowRuntime"]["flowRuntimeId"]
print(f"Pipeline started: {runtime_id}")
```

### Example JSON response

```json
{
  "flowRuntime": {
    "flowRuntimeId": "b2c3d4e5-2222-3333-4444-555566667777",
    "pipelineName": "Continuous Delivery",
    "projectName": "Online Banking",
    "pipelineRunName": "release-2.3.1",
    "status": "running",
    "start": "2024-06-30T13:00:00.000Z"
  }
}
```

---

## 3. Abort a Pipeline Run

**PUT** `/flowRuntimes/{flowRuntimeId}`

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/flowRuntimes/b2c3d4e5-2222-3333-4444-555566667777" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "abortFlowRuntime",
    "reason": "Version conflict detected in staging"
  }'
```

---

## 4. Get Pipeline Run Status

**GET** `/flowRuntimes`

### Optional query parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | string | Filter by project |
| `pipelineName` | string | Filter by pipeline |
| `status` | string | `running`, `completed`, `waiting` |
| `numObjects` | integer | Max records to return |
| `startIndex` | integer | Pagination offset |

### Python example

```python
result = cdro.get(
    "flowRuntimes",
    projectName="Online Banking",
    pipelineName="Continuous Delivery",
    numObjects=5
)
for rt in result.get("flowRuntime", []):
    print(rt["flowRuntimeId"], rt["status"], rt.get("outcome", "n/a"))
```

---

## 5. Create a Release

**POST** `/projects/{projectName}/releases`

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/releases" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "releaseName": "Q3 Release 2024",
    "description": "Q3 quarterly release",
    "plannedStartDate": "2024-07-01T00:00:00.000Z",
    "plannedEndDate": "2024-09-30T00:00:00.000Z"
  }'
```

---

## 6. Start a Release (PUT action)

**PUT** `/projects/{projectName}/releases/{releaseName}`

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/releases/Q3%20Release%202024" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{"request": "startRelease"}'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
release = "Q3 Release 2024"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/releases/{urllib.parse.quote(release)}"

result = requests.put(url, headers=cdro.headers,
                      json={"request": "startRelease"}, verify=False).json()
print(result["release"]["status"])
```

---

## Notes

- **flowRuntimeId vs jobId:** Pipeline runs produce a `flowRuntimeId`; procedure runs produce a `jobId`. Use the appropriate endpoint for each type.
- **Approval gates:** To approve a gate task during a running pipeline, use `PUT /flowRuntimeStates/{stateId}` with `request=approveGate`.
- **URI encoding:** All path segments with spaces must be percent-encoded (`%20`).
- **Swagger UI:** Full parameter documentation at `https://<cdro-server>/rest/doc/v1.0/`.