# REST API: Projects, Procedures, and Steps

## Overview

Projects are the top-level organizational container in CloudBees CD/RO. Procedures are reusable scripts/workflows defined within a project. Steps are the individual actions inside a procedure.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List all projects | GET | `/projects` |
| Get a project | GET | `/projects/{projectName}` |
| Create a project | POST | `/projects` |
| Update a project | PUT | `/projects/{projectName}` |
| Delete a project | DELETE | `/projects/{projectName}` |
| List procedures | GET | `/projects/{projectName}/procedures` |
| Get a procedure | GET | `/projects/{projectName}/procedures/{procedureName}` |
| Create a procedure | POST | `/projects/{projectName}/procedures` |
| Update a procedure | PUT | `/projects/{projectName}/procedures/{procedureName}` |
| Delete a procedure | DELETE | `/projects/{projectName}/procedures/{procedureName}` |
| List steps | GET | `/projects/{projectName}/procedures/{procedureName}/steps` |
| Get a step | GET | `/projects/{projectName}/procedures/{procedureName}/steps/{stepName}` |
| Create a step | POST | `/projects/{projectName}/procedures/{procedureName}/steps` |
| Update a step | PUT | `/projects/{projectName}/procedures/{procedureName}/steps/{stepName}` |
| Delete a step | DELETE | `/projects/{projectName}/procedures/{procedureName}/steps/{stepName}` |
| Run a procedure | PUT | `/projects/{projectName}/procedures/{procedureName}` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `My Project` → `My%20Project`.

---

## 1. List Projects

**GET** `/projects`

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/projects" \
  -H "sessionid: <API-token>"
```

### Python example

```python
result = cdro.get("projects")
for proj in result["project"]:
    print(proj["projectName"], proj["projectId"])
```

### Example JSON response

```json
{
  "project": [
    {
      "projectId": "e1a2b3c4-...",
      "projectName": "Default",
      "description": "Default project",
      "createTime": "2024-01-10T08:00:00.000Z",
      "lastModifiedBy": "admin"
    },
    {
      "projectId": "f5d6e7f8-...",
      "projectName": "Online Banking",
      "description": "Banking app delivery project",
      "createTime": "2024-02-15T09:30:00.000Z",
      "lastModifiedBy": "devops"
    }
  ]
}
```

---

## 2. Create a Project

**POST** `/projects`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | string | Name of the new project |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `credentialName` | string | Default credential for procedures |
| `resourceName` | string | Default resource for procedures |
| `workspaceName` | string | Default workspace |
| `tracked` | boolean | Whether to track project in SCM |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "projectName": "Online Banking",
    "description": "Banking application delivery"
  }'
```

### Python example

```python
result = cdro.post(
    "projects",
    projectName="Online Banking",
    description="Banking application delivery"
)
print(result["project"]["projectId"])
```

---

## 3. Create a Procedure

**POST** `/projects/{projectName}/procedures`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `procedureName` | string | Name of the new procedure |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Procedure description |
| `resourceName` | string | Default resource override |
| `timeLimit` | string | Max allowed run time (e.g., `"3600"`) |
| `timeLimitUnits` | string | `seconds` / `minutes` / `hours` |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/procedures" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "procedureName": "Deploy App",
    "description": "Deploys the banking application",
    "resourceName": "local",
    "timeLimit": "3600",
    "timeLimitUnits": "seconds"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/procedures",
    procedureName="Deploy App",
    description="Deploys the banking application",
    resourceName="local",
    timeLimit="3600",
    timeLimitUnits="seconds"
)
print(result["procedure"]["procedureId"])
```

---

## 4. Create a Step

**POST** `/projects/{projectName}/procedures/{procedureName}/steps`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `stepName` | string | Name for the step |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `command` | string | Shell command to execute |
| `shell` | string | Shell interpreter (e.g., `bash`, `sh`, `ec-perl`) |
| `description` | string | Step description |
| `resourceName` | string | Specific resource to run on |
| `condition` | string | Run condition expression |
| `subprocedure` | string | Subprocedure to call instead of running a command |
| `errorHandling` | string | `abortProcedure` (default) or `stopRunning` |
| `alwaysRun` | boolean | Run even if earlier steps fail |
| `parallel` | boolean | Run this step in parallel with others |
| `timeLimit` | string | Max allowed run time |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/procedures/Deploy%20App/steps" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "stepName": "Run Tests",
    "command": "cd /opt/app && ./run-tests.sh",
    "shell": "bash",
    "description": "Execute unit tests",
    "resourceName": "build-agent-01",
    "errorHandling": "abortProcedure"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
procedure = "Deploy App"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/procedures/{urllib.parse.quote(procedure)}/steps",
    stepName="Run Tests",
    command="cd /opt/app && ./run-tests.sh",
    shell="bash",
    resourceName="build-agent-01",
    errorHandling="abortProcedure"
)
print(result["step"]["stepId"])
```

---

## 5. Run a Procedure (PUT action)

**PUT** `/projects/{projectName}/procedures/{procedureName}`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runProcedure` |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `actualParameter` | array | Runtime parameter overrides (`[{actualParameterName, value}]`) |
| `priority` | string | Job priority: `low`, `normal`, `high`, `highest` |
| `resourceName` | string | Override default resource |

### curl example (with actual parameters)

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/procedures/Deploy%20App" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "runProcedure",
    "actualParameter": [
      {"actualParameterName": "environment", "value": "staging"},
      {"actualParameterName": "version", "value": "2.3.1"}
    ],
    "priority": "high"
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
procedure = "Deploy App"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/procedures/{urllib.parse.quote(procedure)}"

result = requests.put(url, headers=cdro.headers, json={
    "request": "runProcedure",
    "actualParameter": [
        {"actualParameterName": "environment", "value": "staging"},
        {"actualParameterName": "version", "value": "2.3.1"}
    ],
    "priority": "high"
}, verify=False).json()
job_id = result["job"]["jobId"]
print(f"Job started: {job_id}")
```

### Example JSON response

```json
{
  "job": {
    "jobId": "c3d4e5f6-7890-12ab-cdef-34567890abcd",
    "jobName": "Deploy App - 2024-06-30",
    "procedureName": "Deploy App",
    "projectName": "Online Banking",
    "status": "running",
    "outcome": "none",
    "createTime": "2024-06-30T12:15:00.000Z"
  }
}
```

---

## Notes

- **Action pattern:** PUT with `request=<action>` is used for all procedure and object state transitions. The action can be passed as a query parameter or in the request body JSON.
- **URI encoding:** All path segments containing spaces must be percent-encoded. For Python, use `urllib.parse.quote("My Project")` → `My%20Project`.
- **Pagination:** Use `numObjects` and `startIndex` query params on list endpoints to page through large result sets.
- **Swagger UI:** The full parameter reference for all endpoints is at `https://<cdro-server>/rest/doc/v1.0/`.