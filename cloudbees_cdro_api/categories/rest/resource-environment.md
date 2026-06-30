# REST API: Resources and Environments

## Overview

Resources represent the agents (servers, machines, or containers) that execute work in CloudBees CD/RO. Resource pools group resources for load balancing. Environments map logical deployment targets to physical resources via environment tiers. This file covers creating and managing resources, pools, and environments.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List resources | GET | `/resources` |
| Get a resource | GET | `/resources/{resourceName}` |
| Create a resource | POST | `/resources` |
| Update a resource | PUT | `/resources/{resourceName}` |
| Delete a resource | DELETE | `/resources/{resourceName}` |
| Ping a resource | PUT | `/resources/{resourceName}` |
| List resource pools | GET | `/resourcePools` |
| Get a resource pool | GET | `/resourcePools/{resourcePoolName}` |
| Create a resource pool | POST | `/resourcePools` |
| Update a resource pool | PUT | `/resourcePools/{resourcePoolName}` |
| Delete a resource pool | DELETE | `/resourcePools/{resourcePoolName}` |
| List environments | GET | `/projects/{projectName}/environments` |
| Get an environment | GET | `/projects/{projectName}/environments/{environmentName}` |
| Create an environment | POST | `/projects/{projectName}/environments` |
| Update an environment | PUT | `/projects/{projectName}/environments/{environmentName}` |
| Delete an environment | DELETE | `/projects/{projectName}/environments/{environmentName}` |
| List environment tiers | GET | `/projects/{projectName}/environments/{environmentName}/environmentTiers` |
| Get an environment tier | GET | `/projects/{projectName}/environments/{environmentName}/environmentTiers/{tierName}` |
| Create an environment tier | POST | `/projects/{projectName}/environments/{environmentName}/environmentTiers` |
| Add resource to tier | POST | `/projects/{projectName}/environments/{environmentName}/environmentTiers/{tierName}/resources` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `build agent 01` → `build%20agent%2001`.

---

## 1. Create a Resource

**POST** `/resources`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `resourceName` | string | Unique name for the resource |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `hostName` | string | Hostname or IP of the agent machine |
| `port` | integer | Agent port (default: `7800`) |
| `resourcePoolName` | string | Pool to add this resource to |
| `workspaceName` | string | Default workspace for this resource |
| `stepLimit` | integer | Max concurrent steps |
| `enabled` | boolean | Whether the resource is active (default: `true`) |
| `shell` | string | Default shell (`bash`, `sh`, `cmd`, etc.) |
| `agentProtocol` | string | `https` or `local` |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/resources" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "resourceName": "build-agent-01",
    "description": "Primary build agent",
    "hostName": "build01.internal.example.com",
    "port": 7800,
    "stepLimit": 4,
    "shell": "bash"
  }'
```

### Python example

```python
result = cdro.post(
    "resources",
    resourceName="build-agent-01",
    description="Primary build agent",
    hostName="build01.internal.example.com",
    port=7800,
    stepLimit=4,
    shell="bash"
)
print(result["resource"]["resourceId"])
```

### Example JSON response

```json
{
  "resource": {
    "resourceId": "a1b2c3d4-1111-aaaa-bbbb-cccc00001234",
    "resourceName": "build-agent-01",
    "hostName": "build01.internal.example.com",
    "port": "7800",
    "stepLimit": "4",
    "shell": "bash",
    "enabled": "1",
    "agentState": "alive",
    "createTime": "2024-06-30T12:00:00.000Z"
  }
}
```

---

## 2. Ping a Resource (PUT action)

**PUT** `/resources/{resourceName}`

Sends a health-check ping to a resource to verify connectivity.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `ping` |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/resources/build-agent-01" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{"request": "ping"}'
```

### Python example

```python
import requests

url = f"{cdro.base}/resources/build-agent-01"
result = requests.put(url, headers=cdro.headers,
                      json={"request": "ping"}, verify=False).json()
print(result["resource"]["agentState"])
```

### Example JSON response

```json
{
  "resource": {
    "resourceId": "a1b2c3d4-1111-aaaa-bbbb-cccc00001234",
    "resourceName": "build-agent-01",
    "agentState": "alive",
    "agentVersion": "10.2.1.20001",
    "lastPingTime": "2024-06-30T13:00:00.000Z"
  }
}
```

---

## 3. Create a Resource Pool

**POST** `/resourcePools`

Resource pools group multiple resources and enable load-balanced job assignment.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `resourcePoolName` | string | Name for the resource pool |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Pool description |
| `enabled` | boolean | Whether pool is active (default: `true`) |
| `orderingFilter` | string | Resource selection ordering expression |
| `resourceNames` | array | List of resource names to include initially |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/resourcePools" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "resourcePoolName": "build-agents",
    "description": "Pool of build agents",
    "resourceNames": ["build-agent-01", "build-agent-02"]
  }'
```

### Python example

```python
import requests

result = requests.post(
    f"{cdro.base}/resourcePools",
    headers=cdro.headers,
    json={
        "resourcePoolName": "build-agents",
        "description": "Pool of build agents",
        "resourceNames": ["build-agent-01", "build-agent-02"]
    },
    verify=False
).json()
print(result["resourcePool"]["resourcePoolId"])
```

### Example JSON response

```json
{
  "resourcePool": {
    "resourcePoolId": "b2c3d4e5-2222-bbbb-cccc-dddd00002345",
    "resourcePoolName": "build-agents",
    "description": "Pool of build agents",
    "enabled": "1",
    "createTime": "2024-06-30T12:10:00.000Z"
  }
}
```

---

## 4. Create an Environment

**POST** `/projects/{projectName}/environments`

Environments are logical deployment targets (e.g., Dev, Staging, Production). They contain tiers, which map to resources.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `environmentName` | string | Name of the environment |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Environment description |
| `resourceName` | string | Default resource |
| `projectName` | string | Project name (from URL path) |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/environments" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "environmentName": "Staging",
    "description": "Pre-production staging environment"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/environments",
    environmentName="Staging",
    description="Pre-production staging environment"
)
print(result["environment"]["environmentId"])
```

### Example JSON response

```json
{
  "environment": {
    "environmentId": "c3d4e5f6-3333-cccc-dddd-eeee00003456",
    "environmentName": "Staging",
    "projectName": "Online Banking",
    "description": "Pre-production staging environment",
    "createTime": "2024-06-30T12:15:00.000Z"
  }
}
```

---

## 5. Add a Resource to an Environment Tier

Creating an environment tier and adding a resource to it are typically separate steps.

### Step 1: Create a Tier

**POST** `/projects/{projectName}/environments/{environmentName}/environmentTiers`

| Parameter | Type | Description |
|---|---|---|
| `environmentTierName` | string | Name of the tier (required) |
| `description` | string | Tier description (optional) |

### curl example — create tier

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/environments/Staging/environmentTiers" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "environmentTierName": "App Servers",
    "description": "Tomcat application server tier"
  }'
```

### Step 2: Add Resource to Tier

**POST** `/projects/{projectName}/environments/{environmentName}/environmentTiers/{tierName}/resources`

| Parameter | Type | Description |
|---|---|---|
| `resourceName` | string | Name of the resource to add (required) |

### curl example — add resource to tier

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/environments/Staging/environmentTiers/App%20Servers/resources" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{"resourceName": "build-agent-01"}'
```

### Python example — full flow

```python
import urllib.parse

project = "Online Banking"
environment = "Staging"
tier = "App Servers"

# Create tier
cdro.post(
    f"projects/{urllib.parse.quote(project)}/environments/{urllib.parse.quote(environment)}/environmentTiers",
    environmentTierName=tier,
    description="Tomcat application server tier"
)

# Add resource to tier
cdro.post(
    f"projects/{urllib.parse.quote(project)}/environments/{urllib.parse.quote(environment)}/environmentTiers/{urllib.parse.quote(tier)}/resources",
    resourceName="build-agent-01"
)
print("Resource added to tier")
```

### Example JSON response (add resource)

```json
{
  "environmentTier": {
    "environmentTierId": "d4e5f6a7-4444-dddd-eeee-ffff00004567",
    "environmentTierName": "App Servers",
    "environmentName": "Staging",
    "projectName": "Online Banking",
    "resource": [
      {
        "resourceName": "build-agent-01",
        "resourceId": "a1b2c3d4-1111-aaaa-bbbb-cccc00001234"
      }
    ]
  }
}
```

---

## Notes

- **Resource vs resource pool:** Individual resources are used for predictable assignment; resource pools let the system pick available resources dynamically for load balancing.
- **`local` resource:** CloudBees CD/RO always has a built-in resource named `local` that refers to the CD/RO server itself. Use it for quick testing without a dedicated agent.
- **Agent connectivity:** After creating a resource, use the `ping` action to verify the agent is reachable before assigning it to environments or procedures.
- **Environment tiers and components:** When deploying an application, CloudBees maps application components to environment tiers by component/tier name matching. Tier names should match component names or be explicitly mapped.
- **URI encoding:** All path segments with spaces must be percent-encoded. Python: `urllib.parse.quote("App Servers")`.
- **Swagger UI:** Full parameter reference at `https://<cdro-server>/rest/doc/v1.0/`.