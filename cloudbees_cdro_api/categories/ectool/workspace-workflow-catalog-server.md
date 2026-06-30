# ectool — Workspace, Workflow, Catalog & Server

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
| `createWorkspace` | Create a workspace definition | `workspaceName` |
| `modifyWorkspace` | Modify a workspace | `workspaceName` |
| `deleteWorkspace` | Delete a workspace | `workspaceName` |
| `getWorkspace` | Get workspace details | `workspaceName` |
| `getWorkspaces` | List all workspaces | _(none)_ |
| `createGateway` | Create a gateway resource | `gatewayName` |
| `modifyGateway` | Modify a gateway | `gatewayName` |
| `deleteGateway` | Delete a gateway | `gatewayName` |
| `getGateway` | Get gateway details | `gatewayName` |
| `getGateways` | List all gateways | _(none)_ |
| `createZone` | Create a zone | `zoneName` |
| `modifyZone` | Modify a zone | `zoneName` |
| `deleteZone` | Delete a zone | `zoneName` |
| `getZone` | Get zone details | `zoneName` |
| `getZones` | List all zones | _(none)_ |
| `createWorkflow` | Create a workflow definition | `projectName` `workflowName` |
| `runWorkflow` | Run a workflow | `projectName` `workflowName` |
| `getWorkflow` | Get workflow details | `projectName` `workflowName` |
| `createTrigger` | Create a trigger | `projectName` `triggerName` |
| `modifyTrigger` | Modify a trigger | `projectName` `triggerName` |
| `deleteTrigger` | Delete a trigger | `projectName` `triggerName` |
| `getTrigger` | Get trigger details | `projectName` `triggerName` |
| `createCatalog` | Create a catalog | `projectName` `catalogName` |
| `modifyCatalog` | Modify a catalog | `projectName` `catalogName` |
| `deleteCatalog` | Delete a catalog | `projectName` `catalogName` |
| `getCatalog` | Get catalog details | `projectName` `catalogName` |
| `createCatalogItem` | Create an item in a catalog | `projectName` `catalogName` `catalogItemName` |
| `runCatalogItem` | Run a catalog item | `projectName` `catalogName` `catalogItemName` |
| `evalDsl` | Evaluate DSL code | _(none)_ |
| `findObjects` | Search for CD/RO objects | `objectType` |
| `export` | Export CD/RO objects to XML | _(none)_ |
| `import` | Import CD/RO objects from XML | `filePath` |
| `generateDsl` | Generate DSL from existing objects | _(none)_ |
| `getServerStatus` | Get server health/status | _(none)_ |
| `getVersions` | Get server version information | _(none)_ |
| `setServerSetting` | Set a server configuration setting | `settingName` `value` |
| `getServerSetting` | Get a server configuration setting | `settingName` |
| `completeWorkflow` | Mark a workflow instance as completed | `projectName` `workflowName` |
| `transitionWorkflow` | Fire a manual transition in a workflow | _(none — all named)_ |
| `getWorkflows` | List all workflow instances in a project | _(none — all named)_ |
| `deleteWorkflow` | Delete a workflow instance | `projectName` `workflowName` |
| `createWorkflowDefinition` | Create a workflow definition template | `projectName` `workflowDefinitionName` |
| `getWorkflowDefinition` | Get a workflow definition | `projectName` `workflowDefinitionName` |
| `getWorkflowDefinitions` | List all workflow definitions | _(none — all named)_ |
| `modifyWorkflowDefinition` | Modify a workflow definition | `projectName` `workflowDefinitionName` |
| `deleteWorkflowDefinition` | Delete a workflow definition | `projectName` `workflowDefinitionName` |
| `createStateDefinition` | Create a state in a workflow definition | `projectName` `workflowDefinitionName` `stateDefinitionName` |
| `getStateDefinition` | Get a state definition | _(none — all named)_ |
| `getStateDefinitions` | List all state definitions | _(none — all named)_ |
| `modifyStateDefinition` | Modify a state definition | _(none — all named)_ |
| `deleteStateDefinition` | Delete a state definition | _(none — all named)_ |
| `createTransitionDefinition` | Create a transition between states | `projectName` `workflowDefinitionName` `stateDefinitionName` `transitionDefinitionName` `targetState` |
| `getTransitionDefinition` | Get a transition definition | _(none — all named)_ |
| `getTransitionDefinitions` | List transitions in a state | _(none — all named)_ |
| `modifyTransitionDefinition` | Modify a transition | _(none — all named)_ |
| `deleteTransitionDefinition` | Delete a transition | _(none — all named)_ |
| `moveTransitionDefinition` | Reorder a transition | _(none — all named)_ |
| `runScmSync` | Run an SCM sync immediately | _(none — all named)_ |
| `getScmSync` | Get an SCM sync object | _(none — all named)_ |
| `getScmSyncs` | List all SCM syncs | _(none — all named)_ |
| `deleteScmSync` | Delete an SCM sync | _(none — all named)_ |
| `modifyScmSync` | Modify an SCM sync | _(none — all named)_ |
| `runTrigger` | Manually fire a trigger | _(none — all named)_ |
| `setupWebhook` | Run webhook setup for a trigger | _(none — all named)_ |
| `processWebhookTrigger` | Process a webhook event | _(none — all named)_ |
| `getTriggerErrorDetails` | List trigger error details | _(none — all named)_ |
| `getCatalogItem` | Get a catalog item | _(none — all named)_ |
| `getCatalogItems` | List all catalog items | _(none — all named)_ |
| `deleteCatalogItem` | Delete a catalog item | _(none — all named)_ |
| `modifyCatalogItem` | Modify a catalog item | _(none — all named)_ |
| `createTag` | Create a global tag | `tagName` |
| `deleteTag` | Delete a tag | `tagName` |
| `getTag` | Get a tag | `tagName` |
| `getTags` | List all tags | _(none)_ |
| `modifyTag` | Modify a tag | `tagName` |
| `tagObject` | Add a tag to an object | _(none — all named)_ |
| `describeObject` | Introspect the schema of an object type | _(none — all named)_ |
| `revert` | Revert an object to a prior revision | _(none — all named)_ |
| `evalScript` | Evaluate a JavaScript expression in server context | _(none — all named)_ |
| `exportPlugin` | Export a plugin to a file | _(none — all named)_ |
| `shutdownServer` | Shut down the CDRO server | _(none)_ |
| `logout` | Log out and invalidate current session | _(none)_ |
| `logStatistic` | Log a custom metric/statistic | _(none — all named)_ |
| `getServerInfo` | Get server networking and port info | _(none)_ |
| `getServerPublicKey` | Retrieve the server's public key | _(none)_ |
| `getServerSettings` | Get settings for a named category | _(none — all named)_ |
| `searchEntityChange` | Search entity change history | _(none — all named)_ |
| `getEntityChange` | Get a single entity change record | _(none — all named)_ |
| `getEntityChangeDetails` | Get details of an entity change | _(none — all named)_ |
| `getRunHierarchy` | Get the hierarchy of a running job/pipeline | _(none — all named)_ |
| `getRunSchedules` | List schedules that are currently running | _(none)_ |
| `sendReportingData` | Send JSON data to DevOps Insight | _(none — all named)_ |
| `getWorkItems` | List work items for a release or pipeline run | _(none — all named)_ |
| `pruneChangeHistory` | Prune old entity change history records | _(none — all named)_ |

---

### `createWorkspace`

Creates a workspace definition — the shared disk space used by agent resources during job execution.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Human-readable description |
| `--agentDrivePath` | string | Agent-side path (Windows UNC or local) |
| `--agentUncPath` | string | UNC path for Windows agents |
| `--agentUnixPath` | string | Unix path for Linux/macOS agents |
| `--local` | boolean | Use local (per-agent) workspace |

**Examples:**

```bash
ectool createWorkspace linux-workspace \
  --description "Shared build workspace for Linux agents" \
  --agentUnixPath /workspace/builds

ectool createWorkspace local-workspace \
  --agentUnixPath /tmp/cdro-workspace \
  --local 1
```

---

### `createGateway` and `createZone`

```bash
ectool createZone aws-us-east-1 --description "AWS us-east-1 region"

ectool createGateway aws-gw-us-east-1 \
  --description "Gateway agent in us-east-1 VPC" \
  --zoneName aws-us-east-1 \
  --resourceName gateway-agent-us-east-1
```

---

### `createTrigger`

Creates a trigger that fires a procedure or pipeline in response to an event.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--procedureName` | string | Procedure to run on trigger |
| `--pipelineName` | string | Pipeline to run on trigger |
| `--triggerType` | string | `cron`, `webhook`, `scm` |
| `--cronExpression` | string | Cron expression (for `cron` type) |
| `--webhookSecret` | string | Secret for webhook validation |
| `--enabled` | boolean | Enable/disable trigger (1/0) |
| `--actualParameter` | string (repeatable) | `name=value` parameters |

**Examples:**

```bash
# Cron trigger for nightly pipeline run
ectool createTrigger MyProject "Nightly Pipeline" \
  --pipelineName "CI-CD Pipeline" \
  --triggerType cron \
  --cronExpression "0 3 * * *" \
  --actualParameter branch=main

# Webhook trigger
ectool createTrigger MyProject "GitHub Push" \
  --procedureName Build \
  --triggerType webhook \
  --webhookSecret "$WEBHOOK_SECRET"
```

---

### `evalDsl`

Evaluates Groovy DSL code against the CD/RO API.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--dslFile` | string | Path to a `.dsl` or `.yaml` DSL file |
| `--dsl` | string | Inline DSL string |
| `--parameter` | string (repeatable) | `name=value` variables available in DSL |

**Examples:**

```bash
ectool evalDsl --dsl '
project "MyProject", {
  procedure "Build", {
    step "compile", {
      command = "mvn clean package"
      shell   = "bash"
    }
  }
}
'

ectool evalDsl \
  --dslFile /path/to/pipeline.dsl \
  --parameter projectName=MyProject \
  --parameter environment=production
```

---

### `findObjects`

Searches for CD/RO objects by type and filter criteria.

**Examples:**

```bash
# Find all running jobs
ectool findObjects job \
  --filter "propertyName:status,operator:equals,operand1:running" \
  --format json | python3 -c "
import sys, json
for obj in json.load(sys.stdin).get('object', []):
    j = obj.get('job', {})
    print(j.get('jobId'), j.get('projectName'))
"

# Find all failed jobs
ectool findObjects job \
  --filter "propertyName:outcome,operator:equals,operand1:error" \
  --numObjects 50 --sort createTime --sortOrder descending --format json
```

---

### `export` and `generateDsl`

```bash
# Export a project to XML
ectool export \
  --projectName MyProject \
  --fileName /backup/MyProject-$(date +%Y%m%d).xml \
  --withACLs 1

# Generate DSL for a project
ectool generateDsl \
  --projectName MyProject \
  --includeChildren 1 \
  --suppressNulls 1 \
  --suppressDefaults 1 > /repo/pipelines/myproject.groovy

# Import objects from XML
ectool import /backup/MyProject-20241001.xml
```

---

### `createCatalog` and `runCatalogItem`

```bash
ectool createCatalog MyProject "DevOps Self-Service" \
  --description "Self-service operations catalog"

ectool createCatalogItem MyProject "DevOps Self-Service" "Deploy to Staging" \
  --pipelineName "CI-CD Pipeline" \
  --description "Deploy any version to the staging environment"

ectool runCatalogItem MyProject "DevOps Self-Service" "Deploy to Staging" \
  --actualParameter version=2.5.1 \
  --actualParameter branch=main
```

---

### `completeWorkflow` and `transitionWorkflow`

```bash
# Complete a workflow instance
ectool completeWorkflow MyProject "Release-Approval-Workflow"

# Fire a transition
ectool transitionWorkflow \
  --projectName MyProject \
  --workflowName "Release-Approval-Workflow" \
  --stateName "Awaiting-Approval" \
  --transitionName "Approve"
```

---

### `createWorkflowDefinition` and `createStateDefinition`

```bash
ectool createWorkflowDefinition MyProject "Release-Approval-WFD" \
  --description "Approval workflow for production releases" \
  --startingState "Pending-Review"

for STATE in "Pending-Review" "In-Testing" "Approved" "Rejected"; do
    ectool createStateDefinition MyProject "Release-Approval-WFD" "$STATE"
done

# Create transitions
ectool createTransitionDefinition MyProject "Release-Approval-WFD" \
  "Pending-Review" "Approve" "Approved" \
  --transitionType MANUAL

ectool createTransitionDefinition MyProject "Release-Approval-WFD" \
  "Pending-Review" "Reject" "Rejected" \
  --transitionType MANUAL
```

---

### `runScmSync`

```bash
ectool runScmSync \
  --projectName MyProject \
  --scmSyncName "GitHub-DSL-Sync"
```

---

### `runTrigger` and `setupWebhook`

```bash
# Manually fire a trigger
ectool runTrigger \
  --projectName MyProject \
  --triggerName "GitHub Push"

# Register webhook with external service
ectool setupWebhook \
  --projectName MyProject \
  --triggerName "GitHub Push"
```

---

### `createTag` and `tagObject`

```bash
ectool createTag production --description "Production resources" --color "#f44336"
ectool createTag staging --description "Staging resources" --color "#ff9800"

# Tag objects
ectool tagObject --tagName production --projectName MyWebApp
ectool tagObject --tagName production --resourceName prod-agent-01
```

---

### `revert`

Reverts an object to a prior revision from change history.

```bash
ectool revert \
  --objectId "pipeline-12345-abcde" \
  --objectType pipeline \
  --revisionNumber 5
```

---

### `searchEntityChange` and `getEntityChangeDetails`

```bash
# Search for pipeline changes in the last hour
ectool searchEntityChange \
  --objectType pipeline \
  --startTime "$(date -u -v-1H +%Y-%m-%dT%H:%M:%SZ)" \
  --format json

# Get full diff of a change
ectool getEntityChangeDetails \
  --entityChangeId "$CHANGE_ID" \
  --format json | python3 -c "
import sys, json
for d in json.load(sys.stdin).get('entityChangeDetail', []):
    print(f\"{d.get('propertyName')}: '{d.get('oldValue')}' -> '{d.get('newValue')}'\"
)"
```

---

### `sendReportingData`

Sends JSON payload to the DevOps Insight reporting system.

```bash
ectool sendReportingData \
  --reportObjectTypeName "build" \
  --payload "{\"projectName\":\"MyWebApp\",\"branch\":\"main\",\"duration\":142,\"outcome\":\"success\"}"
```

---

### `pruneChangeHistory`

```bash
ectool pruneChangeHistory --olderThan 90 --olderThanUnits days
```

---

### `shutdownServer` and `logout`

```bash
# Safe shutdown: verify no jobs running first
RUNNING=$(ectool findObjects job \
  --filter "propertyName:status,operator:equals,operand1:running" \
  --format json | python3 -c "
import sys, json
print(len(json.load(sys.stdin).get('object', [])))
")
if [ "$RUNNING" -eq 0 ]; then
    ectool shutdownServer
fi

# Log out after scripted operations
ectool logout
```

---

## Shell Scripting Patterns

### Backup all projects to DSL

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DIR="/repo/cdro-dsl-backup/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

ectool getProjects --format json | python3 -c "
import sys, json
for p in json.load(sys.stdin).get('project', []):
    if not p['projectName'].startswith('EC-'):
        print(p['projectName'])
" | while read -r PROJECT; do
    echo "Generating DSL for: $PROJECT"
    ectool generateDsl \
      --projectName "$PROJECT" \
      --includeChildren 1 \
      --suppressNulls 1 \
      --suppressDefaults 1 > "$BACKUP_DIR/${PROJECT}.groovy" 2>/dev/null || \
      echo "  Warning: failed to export $PROJECT"
done

echo "Backup complete: $BACKUP_DIR"
```

### Find and clean up stale jobs

```bash
#!/usr/bin/env bash
ectool findObjects job \
  --filter "propertyName:status,operator:equals,operand1:completed" \
  --numObjects 100 \
  --sort createTime --sortOrder ascending \
  --format json | python3 -c "
import sys, json, subprocess
from datetime import datetime, timedelta, timezone

jobs = json.load(sys.stdin).get('object', [])
cutoff = datetime.now(timezone.utc) - timedelta(days=30)

for obj in jobs:
    job = obj.get('job', {})
    jid = job.get('jobId', '')
    ts = job.get('createTime', '')
    try:
        created = datetime.fromisoformat(ts.replace('Z', '+00:00'))
        if created < cutoff:
            print(f'Deleting old job: {jid}')
            subprocess.run(['ectool', 'deleteJob', jid], check=True)
    except Exception as e:
        print(f'Skipping {jid}: {e}')
"
```

### Server health check

```bash
#!/usr/bin/env bash
echo "=== CloudBees CD/RO Server Health ==="
ectool getServerStatus --format json | python3 -c "
import sys, json
status = json.load(sys.stdin).get('serverStatus', {})
print('Status:', status.get('status', 'unknown'))
print('Version:', status.get('version', 'unknown'))
"

echo ""
echo "=== Running Jobs ==="
ectool findObjects job \
  --filter "propertyName:status,operator:equals,operand1:running" \
  --format json | python3 -c "
import sys, json
jobs = json.load(sys.stdin).get('object', [])
print(f'Running jobs: {len(jobs)}')
for obj in jobs[:10]:
    j = obj.get('job', {})
    print(f\"  {j.get('jobId')}: {j.get('jobName', 'N/A')} ({j.get('projectName', '')})\"
)"
```
