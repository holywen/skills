# ectool — Resource & Environment

Fetch docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

---

## Login / Session Setup

```bash
ectool login admin changeme --server cdro.example.com

export ELECTRIC_COMMANDER_SERVER=cdro.example.com
export COMMANDER_SESSIONID=<token>
```

---

## Command Reference Table

| Command | Description | Positional Args (in order) |
|---------|-------------|---------------------------|
| `createResource` | Create an agent resource | `resourceName` |
| `modifyResource` | Modify resource properties | `resourceName` |
| `deleteResource` | Delete a resource | `resourceName` |
| `getResource` | Get resource details | `resourceName` |
| `getResources` | List all resources | _(none)_ |
| `pingResource` | Ping a resource to check connectivity | `resourceName` |
| `pingAllResources` | Ping all resources and update their status | _(none)_ |
| `createResourcePool` | Create a resource pool | `resourcePoolName` |
| `modifyResourcePool` | Modify a resource pool | `resourcePoolName` |
| `deleteResourcePool` | Delete a resource pool | `resourcePoolName` |
| `getResourcePool` | Get resource pool details | `resourcePoolName` |
| `getResourcePools` | List all resource pools | _(none)_ |
| `addResourcesToPool` | Add resources to a pool | `resourcePoolName` |
| `removeResourcesFromPool` | Remove resources from a pool | `resourcePoolName` |
| `createEnvironment` | Create an environment | `projectName` `environmentName` |
| `modifyEnvironment` | Modify an environment | `projectName` `environmentName` |
| `deleteEnvironment` | Delete an environment | `projectName` `environmentName` |
| `getEnvironment` | Get environment details | `projectName` `environmentName` |
| `getEnvironments` | List environments in a project | `projectName` |
| `provisionEnvironment` | Provision a dynamic environment from a template | _(none)_ |
| `tearDownResource` | Tear down a dynamically provisioned resource | _(none)_ |
| `tearDownResourcePool` | Tear down a dynamically provisioned resource pool | _(none)_ |
| `createEnvironmentTier` | Create a tier in an environment | `projectName` `environmentName` `environmentTierName` |
| `modifyEnvironmentTier` | Modify an environment tier | `projectName` `environmentName` `environmentTierName` |
| `deleteEnvironmentTier` | Delete an environment tier | `projectName` `environmentName` `environmentTierName` |
| `addEnvironmentTierResource` | Add a resource to an environment tier | `projectName` `environmentName` `environmentTierName` |
| `removeEnvironmentTierResource` | Remove a resource from a tier | `projectName` `environmentName` `environmentTierName` |
| `getEnvironmentInventory` | Get deployed component inventory | `projectName` `environmentName` |

---

### `createResource`

Creates an agent resource (build/deploy agent).

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Description |
| `--hostName` | string | Hostname or IP of the agent |
| `--port` | int | Agent port (default 7800) |
| `--workspaceName` | string | Default workspace path |
| `--pools` | string (repeatable) | Assign to resource pools |
| `--zoneName` | string | Zone for this resource |

**Examples:**

```bash
# Create a Linux build agent
ectool createResource agent-linux-01 \
  --hostName build-01.internal.example.com \
  --port 7800 \
  --workspaceName /workspace/builds

# Create resource assigned to pools
ectool createResource agent-deploy-01 \
  --hostName deploy-01.internal \
  --pools linux-pool \
  --pools deploy-pool
```

---

### `pingResource`

Pings a resource to verify the agent is reachable and healthy.

**Examples:**

```bash
# Ping a single resource
ectool pingResource agent-linux-01

# Ping with extended timeout
ectool pingResource agent-linux-01 --timeout 120
```

---

### `createResourcePool`

Creates a named pool that groups resources for load balancing.

**Examples:**

```bash
# Create a pool
ectool createResourcePool linux-build-pool \
  --description "Linux build agents" \
  --orderingFilter highestAvailable

# Add resources to the pool
ectool addResourcesToPool linux-build-pool \
  --resourceName agent-linux-01 \
  --resourceName agent-linux-02 \
  --resourceName agent-linux-03

# Use pool in procedure
ectool modifyProcedure MyProject Build \
  --defaultResourceName linux-build-pool
```

---

### `createEnvironment`

Creates a deployment environment (staging, production, etc.).

**Examples:**

```bash
# Create staging environment
ectool createEnvironment MyProject staging \
  --description "Staging environment for QA validation"

# Create production environment
ectool createEnvironment MyProject production \
  --description "Production environment — requires approval"

# List all environments
ectool getEnvironments MyProject --format json | python3 -c "
import sys, json
envs = json.load(sys.stdin).get('environment', [])
for e in envs:
    print(e['environmentName'])
"
```

---

### `createEnvironmentTier`

Creates a tier (logical grouping) within an environment.

**Examples:**

```bash
# Create Web tier
ectool createEnvironmentTier MyProject staging web-tier \
  --description "Web application servers"

# Create Database tier
ectool createEnvironmentTier MyProject staging db-tier \
  --description "Database servers"
```

---

### `addEnvironmentTierResource`

Adds a resource to an environment tier.

**Examples:**

```bash
# Add single resource
ectool addEnvironmentTierResource MyProject staging web-tier \
  --resourceName web-server-01

# Add multiple resources to a tier
ectool addEnvironmentTierResource MyProject production app-tier \
  --resourceName app-server-01
ectool addEnvironmentTierResource MyProject production app-tier \
  --resourceName app-server-02
```

---

## Shell Scripting Patterns

### Full environment scaffold

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT="MyProject"

for ENV in staging uat production; do
    ectool createEnvironment "$PROJECT" "$ENV" \
      --description "$ENV environment"

    ectool createEnvironmentTier "$PROJECT" "$ENV" web-tier
    ectool createEnvironmentTier "$PROJECT" "$ENV" app-tier
    ectool createEnvironmentTier "$PROJECT" "$ENV" db-tier

    echo "Created environment: $ENV"
done

ectool addEnvironmentTierResource MyProject staging web-tier \
  --resourceName staging-web-01
ectool addEnvironmentTierResource MyProject staging app-tier \
  --resourceName staging-app-01
ectool addEnvironmentTierResource MyProject staging db-tier \
  --resourceName staging-db-01

echo "Environment scaffold complete."
```
