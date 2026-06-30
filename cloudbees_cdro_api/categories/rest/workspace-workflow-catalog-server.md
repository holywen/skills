# REST API: Workspaces, Workflows, Catalogs, and Server Operations

## Overview

Workspaces define shared filesystem paths used by agents during job execution. Workflow definitions model state-machine-style automation flows. Catalogs provide a service catalog of self-service items that end users can run. The server endpoint exposes administrative operations such as DSL evaluation and system export/import. This file covers creating and using all four, plus key server-level actions.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List workspaces | GET | `/workspaces` |
| Get a workspace | GET | `/workspaces/{workspaceName}` |
| Create a workspace | POST | `/workspaces` |
| Update a workspace | PUT | `/workspaces/{workspaceName}` |
| Delete a workspace | DELETE | `/workspaces/{workspaceName}` |
| List workflow definitions | GET | `/projects/{projectName}/workflowDefinitions` |
| Get a workflow definition | GET | `/projects/{projectName}/workflowDefinitions/{workflowName}` |
| Create a workflow definition | POST | `/projects/{projectName}/workflowDefinitions` |
| List state definitions | GET | `/projects/{projectName}/workflowDefinitions/{workflowName}/stateDefinitions` |
| Create a state definition | POST | `/projects/{projectName}/workflowDefinitions/{workflowName}/stateDefinitions` |
| Run a workflow | PUT | `/projects/{projectName}/workflowDefinitions/{workflowName}` |
| List workflows (instances) | GET | `/projects/{projectName}/workflows` |
| Get a workflow instance | GET | `/projects/{projectName}/workflows/{workflowId}` |
| List catalogs | GET | `/projects/{projectName}/catalogs` |
| Get a catalog | GET | `/projects/{projectName}/catalogs/{catalogName}` |
| Create a catalog | POST | `/projects/{projectName}/catalogs` |
| List catalog items | GET | `/projects/{projectName}/catalogs/{catalogName}/catalogItems` |
| Create a catalog item | POST | `/projects/{projectName}/catalogs/{catalogName}/catalogItems` |
| Run a catalog item | PUT | `/projects/{projectName}/catalogs/{catalogName}/catalogItems/{itemName}` |
| Get server status | GET | `/server` |
| Evaluate DSL | PUT | `/server` |
| Export objects | PUT | `/server` |
| Import objects | PUT | `/server` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `My Workspace` → `My%20Workspace`.

---

## 1. Create a Workspace

**POST** `/workspaces`

Workspaces define a directory path on the agent where job artifacts and working files are placed during execution.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `workspaceName` | string | Name for the workspace |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `agentDrivePath` | string | Path on Windows agents |
| `agentUncPath` | string | UNC path for Windows agents |
| `agentUnixPath` | string | Path on Unix/Linux agents |
| `local` | boolean | If `true`, workspace is local to each agent |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/workspaces" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "workspaceName": "build-workspace",
    "description": "Shared build workspace for CI jobs",
    "agentUnixPath": "/opt/cdro/workspace",
    "agentDrivePath": "C:/cdro/workspace"
  }'
```

### Python example

```python
result = cdro.post(
    "workspaces",
    workspaceName="build-workspace",
    description="Shared build workspace for CI jobs",
    agentUnixPath="/opt/cdro/workspace",
    agentDrivePath="C:/cdro/workspace"
)
print(result["workspace"]["workspaceId"])
```

### Example JSON response

```json
{
  "workspace": {
    "workspaceId": "a1b2c3d4-aaaa-1111-2222-333344440001",
    "workspaceName": "build-workspace",
    "description": "Shared build workspace for CI jobs",
    "agentUnixPath": "/opt/cdro/workspace",
    "agentDrivePath": "C:/cdro/workspace",
    "local": "0",
    "createTime": "2024-06-30T12:00:00.000Z"
  }
}
```

---

## 2. Run a Workflow (PUT action)

**PUT** `/projects/{projectName}/workflowDefinitions/{workflowName}`

Starts a new instance of a workflow definition. Returns a `workflow` object representing the running instance.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runWorkflow` |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `startingState` | string | State definition to begin execution in |
| `actualParameter` | array | Runtime parameters as `[{actualParameterName, value}]` |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/workflowDefinitions/Approval%20Flow" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "runWorkflow",
    "startingState": "Submit for Review",
    "actualParameter": [
      {"actualParameterName": "changeTicket", "value": "CHG-1234"},
      {"actualParameterName": "requestor", "value": "jdoe"}
    ]
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
workflow = "Approval Flow"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/workflowDefinitions/{urllib.parse.quote(workflow)}"

result = requests.put(url, headers=cdro.headers, json={
    "request": "runWorkflow",
    "startingState": "Submit for Review",
    "actualParameter": [
        {"actualParameterName": "changeTicket", "value": "CHG-1234"},
        {"actualParameterName": "requestor", "value": "jdoe"}
    ]
}, verify=False).json()

workflow_id = result["workflow"]["workflowId"]
print(f"Workflow started: {workflow_id}")
```

### Example JSON response

```json
{
  "workflow": {
    "workflowId": "b2c3d4e5-bbbb-2222-3333-444455550002",
    "workflowDefinitionName": "Approval Flow",
    "projectName": "Online Banking",
    "activeState": "Submit for Review",
    "status": "active",
    "start": "2024-06-30T13:00:00.000Z"
  }
}
```

---

## 3. Run a Catalog Item (PUT action)

**PUT** `/projects/{projectName}/catalogs/{catalogName}/catalogItems/{itemName}`

Catalog items are self-service actions end users can run from the CloudBees CD/RO service catalog. Running one creates a job or pipeline run.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runCatalogItem` |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `actualParameter` | array | Runtime parameters as `[{actualParameterName, value}]` |

### First: Create Catalog and Catalog Item

**POST** `/projects/{projectName}/catalogs`

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/catalogs" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "catalogName": "Self Service",
    "description": "Developer self-service catalog"
  }'
```

**POST** `/projects/{projectName}/catalogs/{catalogName}/catalogItems`

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/catalogs/Self%20Service/catalogItems" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "catalogItemName": "Deploy to Dev",
    "description": "Self-service deploy to dev environment",
    "procedureName": "Deploy App",
    "projectName": "Online Banking"
  }'
```

### curl example — run catalog item

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/catalogs/Self%20Service/catalogItems/Deploy%20to%20Dev" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "runCatalogItem",
    "actualParameter": [
      {"actualParameterName": "version", "value": "2.3.1"}
    ]
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
catalog = "Self Service"
item = "Deploy to Dev"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/catalogs/{urllib.parse.quote(catalog)}/catalogItems/{urllib.parse.quote(item)}"

result = requests.put(url, headers=cdro.headers, json={
    "request": "runCatalogItem",
    "actualParameter": [
        {"actualParameterName": "version", "value": "2.3.1"}
    ]
}, verify=False).json()
print(result["job"]["jobId"])
```

### Example JSON response

```json
{
  "job": {
    "jobId": "c3d4e5f6-cccc-3333-4444-555566660003",
    "jobName": "Deploy App - 2024-06-30",
    "status": "running",
    "outcome": "none",
    "projectName": "Online Banking",
    "start": "2024-06-30T14:00:00.000Z"
  }
}
```

---

## 4. Evaluate DSL (PUT server action)

**PUT** `/server`

The DSL evaluation endpoint accepts a Groovy DSL string and executes it server-side. This is a powerful administrative action that can create, modify, or delete any object.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `evalDsl` |
| `dsl` | string | Groovy DSL code to evaluate |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `parameters` | object | Key-value map of parameters to pass into the DSL |
| `dslFile` | string | Path to a DSL file on the server (alternative to inline `dsl`) |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/server" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "evalDsl",
    "dsl": "project(\\"DSL Test Project\\") { description \\"Created via REST DSL eval\\" }"
  }'
```

### curl example — DSL with parameters

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/server" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "evalDsl",
    "dsl": "project(args.projectName) { description args.description }",
    "parameters": {
      "projectName": "My Dynamic Project",
      "description": "Created with dynamic DSL"
    }
  }'
```

### Python example

```python
import requests

dsl_code = '''
project("REST API Demo") {
  description "Demonstrating DSL via REST"
  procedure("Hello World") {
    step("say hello") {
      command "echo Hello from DSL"
      shell "bash"
    }
  }
}
'''

result = requests.put(
    f"{cdro.base}/server",
    headers=cdro.headers,
    json={"request": "evalDsl", "dsl": dsl_code},
    verify=False
).json()
print(result)
```

### Example JSON response

```json
{
  "requestId": "d4e5f6a7-dddd-4444-5555-666677770004",
  "status": "completed",
  "result": {
    "project": {
      "projectName": "REST API Demo",
      "projectId": "e5f6a7b8-eeee-5555-6666-777788880005"
    }
  }
}
```

---

## 5. Get Server Status

**GET** `/server`

Returns the server version, license state, and operational status.

### curl example

```bash
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/server" \
  -H "sessionid: <API-token>"
```

### Python example

```python
result = cdro.get("server")
server = result["server"]
print(f"Version: {server['version']}")
print(f"License: {server.get('licenseState', 'unknown')}")
```

### Example JSON response

```json
{
  "server": {
    "serverId": "f6a7b8c9-ffff-6666-7777-888899990006",
    "version": "2024.03.0.171234",
    "releaseVersion": "2024.03",
    "licenseState": "valid",
    "licenseExpiration": "2025-12-31T00:00:00.000Z",
    "startTime": "2024-06-01T08:00:00.000Z",
    "timeZone": "America/New_York",
    "pluginDirectory": "/opt/cloudbees/sda/plugins"
  }
}
```

---

## 6. Export Objects (PUT server action)

**PUT** `/server`

Exports one or more objects to DSL or JSON format for backup or migration.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `export` |
| `path` | string | Server-side path to export to (absolute path on server filesystem) |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | string | Specific project to export |
| `procedureName` | string | Specific procedure to export |
| `exportChildren` | boolean | Include child objects (default: `true`) |
| `suppressDefaults` | boolean | Omit properties at their default values (default: `true`) |
| `format` | string | `dsl` (default) or `json` |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/server" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "export",
    "projectName": "Online Banking",
    "path": "/tmp/export/online-banking.dsl",
    "exportChildren": true,
    "suppressDefaults": true,
    "format": "dsl"
  }'
```

### Python example

```python
import requests

result = requests.put(
    f"{cdro.base}/server",
    headers=cdro.headers,
    json={
        "request": "export",
        "projectName": "Online Banking",
        "path": "/tmp/export/online-banking.dsl",
        "exportChildren": True,
        "suppressDefaults": True,
        "format": "dsl"
    },
    verify=False
).json()
print("Export result:", result)
```

### Example JSON response

```json
{
  "requestId": "a7b8c9d0-aaaa-7777-8888-999900001117",
  "status": "completed",
  "exportedFile": "/tmp/export/online-banking.dsl",
  "objectCount": 47
}
```

---

## Notes

- **DSL eval power and risk:** The `evalDsl` endpoint can create, modify, or delete any object in the system. Restrict API token access to trusted automation accounts only. All DSL eval calls are audit-logged.
- **Workspace paths:** Most jobs use the `default` workspace unless overridden. Create additional workspaces when jobs on different resources need isolated directories.
- **Workflow vs pipeline:** Workflows model long-running approval and state-machine flows (days/weeks). Pipelines model short-lived CI/CD flows (minutes/hours). Use workflows for change-management processes with manual transitions.
- **Catalog items:** Catalog items can wrap procedures, pipelines, or even direct DSL. They are the recommended self-service interface for end users, as they can include icons, descriptions, and parameter forms.
- **Export/import pattern:** Export produces a DSL file on the server filesystem. To retrieve it, use a separate step (e.g., `ectool getProperty` or SSH). For direct DSL string export (no file), use `evalDsl` with a `getDSL` call inside the DSL.
- **URI encoding:** All path segments with spaces must be percent-encoded. Python: `urllib.parse.quote("My Workspace")`.
- **Swagger UI:** Full parameter reference including workflow state transition operations and import options at `https://<cdro-server>/rest/doc/v1.0/`.