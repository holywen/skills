# REST API: Applications and Components

## Overview

Applications in CloudBees CD/RO model deployable software units. Each application contains components (deployable artifacts or configuration units) and application processes (the steps to deploy or undeploy). This file covers creating applications and components, defining processes, and triggering deployments.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List applications | GET | `/projects/{projectName}/applications` |
| Get an application | GET | `/projects/{projectName}/applications/{applicationName}` |
| Create an application | POST | `/projects/{projectName}/applications` |
| Update an application | PUT | `/projects/{projectName}/applications/{applicationName}` |
| Delete an application | DELETE | `/projects/{projectName}/applications/{applicationName}` |
| Deploy an application | PUT | `/projects/{projectName}/applications/{applicationName}` |
| List components | GET | `/projects/{projectName}/applications/{applicationName}/components` |
| Get a component | GET | `/projects/{projectName}/applications/{applicationName}/components/{componentName}` |
| Create a component | POST | `/projects/{projectName}/applications/{applicationName}/components` |
| Update a component | PUT | `/projects/{projectName}/applications/{applicationName}/components/{componentName}` |
| Delete a component | DELETE | `/projects/{projectName}/applications/{applicationName}/components/{componentName}` |
| List app processes | GET | `/projects/{projectName}/applications/{applicationName}/processes` |
| Get an app process | GET | `/projects/{projectName}/applications/{applicationName}/processes/{processName}` |
| Create an app process | POST | `/projects/{projectName}/applications/{applicationName}/processes` |
| List process steps | GET | `/projects/{projectName}/applications/{applicationName}/processes/{processName}/processSteps` |
| Create a process step | POST | `/projects/{projectName}/applications/{applicationName}/processes/{processName}/processSteps` |
| Run an app process | PUT | `/projects/{projectName}/applications/{applicationName}` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `Banking App` → `Banking%20App`.

---

## 1. Create an Application

**POST** `/projects/{projectName}/applications`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `applicationName` | string | Name of the application |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Application description |
| `resourceName` | string | Default resource |
| `workspaceName` | string | Default workspace |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/applications" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "applicationName": "Banking App",
    "description": "Core banking application"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/applications",
    applicationName="Banking App",
    description="Core banking application"
)
print(result["application"]["applicationId"])
```

### Example JSON response

```json
{
  "application": {
    "applicationId": "a1b2c3d4-aaaa-bbbb-cccc-ddddeeee0001",
    "applicationName": "Banking App",
    "projectName": "Online Banking",
    "description": "Core banking application",
    "createTime": "2024-06-30T12:00:00.000Z"
  }
}
```

---

## 2. Create a Component

**POST** `/projects/{projectName}/applications/{applicationName}/components`

Components represent a deployable unit within an application (e.g., a WAR file, a configuration package).

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `componentName` | string | Name of the component |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Component description |
| `componentType` | string | `artifact` (default) or `other` |
| `pluginName` | string | Deploy plugin to use (e.g., `EC-Artifact`) |
| `pluginKey` | string | Plugin key |
| `reference` | boolean | Whether component is a reference to another |
| `applicationName` | string | Parent application (inherited from URL) |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/applications/Banking%20App/components" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "componentName": "webapp-war",
    "description": "Banking web application WAR file",
    "componentType": "artifact"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
app = "Banking App"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/applications/{urllib.parse.quote(app)}/components",
    componentName="webapp-war",
    description="Banking web application WAR file",
    componentType="artifact"
)
print(result["component"]["componentId"])
```

### Example JSON response

```json
{
  "component": {
    "componentId": "b2c3d4e5-bbbb-cccc-dddd-eeeeffffaa02",
    "componentName": "webapp-war",
    "applicationName": "Banking App",
    "projectName": "Online Banking",
    "componentType": "artifact",
    "description": "Banking web application WAR file",
    "createTime": "2024-06-30T12:05:00.000Z"
  }
}
```

---

## 3. Create an Application Process

**POST** `/projects/{projectName}/applications/{applicationName}/processes`

Application processes define the deployment steps for the application. Common process types are `deploy` and `undeploy`.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `processName` | string | Name of the process |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Process description |
| `processType` | string | `deploy` (default), `undeploy`, `other`, `rollback`, `checkForDrift`, `selfService` |
| `componentName` | string | Associated component (for component processes) |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/applications/Banking%20App/processes" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "processName": "Deploy to Tomcat",
    "processType": "deploy",
    "description": "Deploys WAR to Tomcat servers"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
app = "Banking App"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/applications/{urllib.parse.quote(app)}/processes",
    processName="Deploy to Tomcat",
    processType="deploy",
    description="Deploys WAR to Tomcat servers"
)
print(result["process"]["processId"])
```

### Example JSON response

```json
{
  "process": {
    "processId": "c3d4e5f6-cccc-dddd-eeee-ffff00001103",
    "processName": "Deploy to Tomcat",
    "processType": "deploy",
    "applicationName": "Banking App",
    "projectName": "Online Banking",
    "description": "Deploys WAR to Tomcat servers",
    "createTime": "2024-06-30T12:10:00.000Z"
  }
}
```

---

## 4. Create a Process Step

**POST** `/projects/{projectName}/applications/{applicationName}/processes/{processName}/processSteps`

Process steps are actions within an application process, such as deploying a component or running a procedure.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `processStepName` | string | Name of the step |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `processStepType` | string | `component` (default), `procedure`, `manual`, `notificationEmail` |
| `componentName` | string | Component to deploy (for `component` type steps) |
| `subprocedure` | string | Procedure to call (for `procedure` type steps) |
| `subproject` | string | Project containing the subprocedure |
| `description` | string | Step description |
| `errorHandling` | string | `abortJob` or `continueOnError` |
| `condition` | string | Run condition expression |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/applications/Banking%20App/processes/Deploy%20to%20Tomcat/processSteps" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "processStepName": "Deploy WAR",
    "processStepType": "component",
    "componentName": "webapp-war",
    "description": "Deploy the WAR file to Tomcat"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
app = "Banking App"
process = "Deploy to Tomcat"

result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/applications/{urllib.parse.quote(app)}/processes/{urllib.parse.quote(process)}/processSteps",
    processStepName="Deploy WAR",
    processStepType="component",
    componentName="webapp-war",
    description="Deploy the WAR file to Tomcat"
)
print(result["processStep"]["processStepId"])
```

### Example JSON response

```json
{
  "processStep": {
    "processStepId": "d4e5f6a7-dddd-eeee-ffff-0000111122234",
    "processStepName": "Deploy WAR",
    "processName": "Deploy to Tomcat",
    "applicationName": "Banking App",
    "projectName": "Online Banking",
    "processStepType": "component",
    "componentName": "webapp-war",
    "createTime": "2024-06-30T12:15:00.000Z"
  }
}
```

---

## 5. Deploy an Application (PUT action)

**PUT** `/projects/{projectName}/applications/{applicationName}`

Triggers an application deployment by running a process against a specific environment. Uses `request=runProcess`.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `runProcess` |
| `processName` | string | Application process to execute |
| `environmentName` | string | Target environment |
| `environmentProjectName` | string | Project containing the environment (if different) |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `actualParameter` | array | Runtime parameters as `[{actualParameterName, value}]` |
| `snapshotName` | string | Artifact snapshot to deploy |
| `deployAllComponents` | boolean | Deploy all components (default: `true`) |
| `artifactVersionPerComponent` | array | Per-component artifact versions |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/applications/Banking%20App" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "runProcess",
    "processName": "Deploy to Tomcat",
    "environmentName": "Staging",
    "environmentProjectName": "Online Banking",
    "snapshotName": "v2.3.1"
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
app = "Banking App"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/applications/{urllib.parse.quote(app)}"

result = requests.put(url, headers=cdro.headers, json={
    "request": "runProcess",
    "processName": "Deploy to Tomcat",
    "environmentName": "Staging",
    "environmentProjectName": "Online Banking",
    "snapshotName": "v2.3.1"
}, verify=False).json()

job_id = result["job"]["jobId"]
print(f"Deployment started: {job_id}")
```

### Example JSON response

```json
{
  "job": {
    "jobId": "e5f6a7b8-eeee-ffff-0000-111122223335",
    "jobName": "Banking App - Deploy to Tomcat - 2024-06-30",
    "status": "running",
    "outcome": "none",
    "projectName": "Online Banking",
    "applicationName": "Banking App",
    "environmentName": "Staging",
    "start": "2024-06-30T14:00:00.000Z"
  }
}
```

---

## Notes

- **Application vs component processes:** Applications have application-level processes (e.g., "Deploy") that orchestrate component-level steps. Components can also have their own processes for fine-grained control.
- **Snapshots:** Snapshots capture a consistent set of artifact versions and can be referenced by name during deployment to ensure repeatable deploys.
- **Environment requirement:** The `runProcess` action requires `environmentName`. The environment must exist and have resources mapped to its tiers.
- **URI encoding:** All path segments with spaces must be percent-encoded. Python: `urllib.parse.quote("Banking App")` → `Banking%20App`.
- **Swagger UI:** Full parameter reference including artifact version mapping options at `https://<cdro-server>/rest/doc/v1.0/`.