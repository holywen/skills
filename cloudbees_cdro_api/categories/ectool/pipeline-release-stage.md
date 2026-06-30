# ectool — Pipeline, Release & Stage

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
| `createPipeline` | Create a pipeline in a project | `projectName` `pipelineName` |
| `modifyPipeline` | Modify pipeline properties | `projectName` `pipelineName` |
| `deletePipeline` | Delete a pipeline | `projectName` `pipelineName` |
| `getPipeline` | Get pipeline details | `projectName` `pipelineName` |
| `getPipelines` | List pipelines in a project | `projectName` |
| `runPipeline` | Run a pipeline | `projectName` `pipelineName` |
| `abortPipelineRun` | Abort a running pipeline run | `flowRuntimeId` |
| `getPipelineRun` | Get pipeline run details | `flowRuntimeId` |
| `getPipelineRuns` | List pipeline runs | _(none)_ |
| `createStage` | Create a stage in a pipeline | `projectName` `pipelineName` `stageName` |
| `modifyStage` | Modify a stage | `projectName` `pipelineName` `stageName` |
| `deleteStage` | Delete a stage | `projectName` `pipelineName` `stageName` |
| `createTask` | Create a task in a stage | `projectName` `pipelineName` `stageName` `taskName` |
| `modifyTask` | Modify a task | `projectName` `pipelineName` `stageName` `taskName` |
| `deleteTask` | Delete a task | `projectName` `pipelineName` `stageName` `taskName` |
| `completeManualTask` | Complete a manual approval task | `flowRuntimeId` `stageName` `taskName` |
| `createGate` | Create a gate between stages | `projectName` `pipelineName` `stageName` |
| `createRelease` | Create a release | `projectName` `releaseName` |
| `modifyRelease` | Modify a release | `projectName` `releaseName` |
| `deleteRelease` | Delete a release | `projectName` `releaseName` |
| `startRelease` | Start a release | `projectName` `releaseName` |
| `getReleaseRun` | Get release run details | `flowRuntimeId` |
| `abortAllPipelineRuns` | Abort all active pipeline runs in a project | _(none — all named)_ |
| `pausePipelineRun` | Pause an active pipeline run | _(none — all named)_ |
| `resumePipelineRun` | Resume a paused pipeline run | _(none — all named)_ |
| `restartPipelineRun` | Restart a pipeline run from a specified stage | _(none — all named)_ |
| `attachPipelineRun` | Attach a pipeline run to a release | _(none — all named)_ |
| `detachPipelineRun` | Detach a pipeline run from a release | _(none — all named)_ |
| `getAttachedPipelineRuns` | List all pipeline runs attached to a release | _(none — all named)_ |
| `getPipelineRuntimeDetails` | Detailed info for a pipeline run | _(none — all named)_ |
| `getPipelineRunAuditReport` | Audit report for a pipeline run | _(none — all named)_ |
| `setPipelineRunName` | Rename an active pipeline run | _(none — all named)_ |
| `deletePipelineRun` | Delete a completed pipeline run record | _(none — all named)_ |
| `completeRelease` | Mark a release as completed | `projectName` `releaseName` |
| `getRelease` | Get a release definition | `projectName` `releaseName` |
| `getReleaseTimelineDetails` | Return timeline details for a release | _(none — all named)_ |
| `addSubrelease` | Add a subrelease to a release | _(none — all named)_ |
| `getSubrelease` | Get a subrelease | _(none — all named)_ |
| `getSubreleases` | List all subreleases | _(none — all named)_ |
| `removeSubrelease` | Remove a subrelease | _(none — all named)_ |
| `getStage` | Get a pipeline stage definition | _(none — all named)_ |
| `getStages` | List all stages in a pipeline | _(none — all named)_ |
| `getTask` | Get a task definition | _(none — all named)_ |
| `getTasks` | List all tasks in a stage | _(none — all named)_ |
| `retryTask` | Retry a failed task in a pipeline run | _(none — all named)_ |
| `getAllWaitingTasks` | List all tasks waiting for manual action | _(none — all named)_ |
| `getWaitingTasks` | List waiting tasks in a specific stage | _(none — all named)_ |
| `createTaskGroup` | Create a task group container | _(none — all named)_ |
| `removeTaskGroup` | Remove a task group | _(none — all named)_ |
| `runFutureTask` | Manually trigger a future-scheduled task | _(none — all named)_ |
| `getGate` | Get a gate definition | _(none — all named)_ |
| `deleteGate` | Delete a gate | _(none — all named)_ |
| `modifyGate` | Modify a gate | _(none — all named)_ |

---

### `createPipeline`

Creates a pipeline definition in a project.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Owning project |
| `pipelineName` | Yes | Unique pipeline name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Human-readable description |
| `--resourceName` | string | Default resource |
| `--enabled` | boolean | Whether pipeline is enabled (1/0) |

**Examples:**

```bash
ectool createPipeline MyWebApp "CI-CD Pipeline" \
  --description "Build, test, and deploy MyWebApp" \
  --resourceName agent-linux-01

ectool getPipelines MyWebApp --format json | python3 -c "
import sys, json
for p in json.load(sys.stdin).get('pipeline', []):
    print(p['pipelineName'])
"
```

---

### `createStage`

Creates a stage within a pipeline.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `pipelineName` | Yes | Pipeline name |
| `stageName` | Yes | Stage name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Stage description |
| `--resourceName` | string | Default resource for tasks |
| `--colorCode` | string | Color for UI display (hex) |

**Examples:**

```bash
ectool createStage MyWebApp "CI-CD Pipeline" Build \
  --description "Compile and unit test"

ectool createStage MyWebApp "CI-CD Pipeline" Test \
  --colorCode "#ffc107"

ectool createStage MyWebApp "CI-CD Pipeline" "Deploy-Production" \
  --colorCode "#f44336"
```

---

### `createTask`

Creates a task within a stage.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `pipelineName` | Yes | Pipeline name |
| `stageName` | Yes | Stage name |
| `taskName` | Yes | Task name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--taskType` | string | `PROCEDURE`, `PROCESS`, `MANUAL`, `APPROVAL`, `PLUGIN`, `DEPLOYER`, `COMMAND` |
| `--subprocedure` | string | Procedure name (for `PROCEDURE` type) |
| `--subproject` | string | Project for subprocedure |
| `--command` | string | Shell command (for `COMMAND` type) |
| `--applicationName` | string | Application name (for `PROCESS` type) |
| `--processName` | string | Process name (for `PROCESS` type) |
| `--environmentName` | string | Target environment |
| `--instruction` | string | Instructions shown to approver (for `MANUAL`) |
| `--actualParameter` | string (repeatable) | `name=value` parameters |

**Examples:**

```bash
# Task calling a procedure
ectool createTask MyWebApp "CI-CD Pipeline" Build "Run Maven Build" \
  --taskType PROCEDURE \
  --subproject MyWebApp \
  --subprocedure Build \
  --actualParameter branch=main

# Manual approval task
ectool createTask MyWebApp "CI-CD Pipeline" "Deploy-Production" "Approval" \
  --taskType MANUAL \
  --instruction "Review test results and approve production deployment."

# Deploy application task
ectool createTask MyWebApp "CI-CD Pipeline" "Deploy-Staging" "Deploy App" \
  --taskType PROCESS \
  --applicationName MyWebApp \
  --processName Deploy \
  --environmentName staging \
  --environmentProjectName MyWebApp
```

---

### `runPipeline`

Submits a pipeline for execution. Returns a `flowRuntimeId`.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `pipelineName` | Yes | Pipeline to run |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--actualParameter` | string (repeatable) | `name=value` run parameters |
| `--startingStage` | string | Start from a specific stage |

**Examples:**

```bash
# Run with parameters and capture flowRuntimeId
FLOW_ID=$(ectool runPipeline MyWebApp "CI-CD Pipeline" \
  --actualParameter branch=main \
  --format json | python3 -c "
import sys, json
print(json.load(sys.stdin)['flowRuntime']['flowRuntimeId'])
")

# Poll for status
while true; do
    STATUS=$(ectool getPipelineRun "$FLOW_ID" --format json | python3 -c "
import sys, json
print(json.load(sys.stdin)['flowRuntime'].get('status', 'unknown'))
")
    [[ "$STATUS" =~ ^(completed|failed|aborted)$ ]] && break
    sleep 15
done
```

---

### `completeManualTask`

Approves or rejects a manual/approval task in a running pipeline.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `flowRuntimeId` | Yes | Pipeline run containing the task |
| `stageName` | Yes | Stage containing the task |
| `taskName` | Yes | Name of the manual task |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--action` | string | `approve` or `reject` |
| `--comment` | string | Approval/rejection comment |

**Examples:**

```bash
ectool completeManualTask "$FLOW_ID" "Deploy-Production" "Approval" \
  --action approve \
  --comment "Test results look good. Approved."

ectool completeManualTask "$FLOW_ID" "Deploy-Production" "Approval" \
  --action reject \
  --comment "Regression detected. Rejecting."
```

---

### `createRelease` and `startRelease`

**Examples:**

```bash
ectool createRelease MyWebApp "Release-2.5.0" \
  --description "Q3 2024 feature release" \
  --plannedStartDate "2024-10-01" \
  --plannedEndDate "2024-10-15" \
  --pipelineName "CI-CD Pipeline"

FLOW_ID=$(ectool startRelease MyWebApp "Release-2.5.0" \
  --format json | python3 -c "
import sys, json
print(json.load(sys.stdin)['flowRuntime']['flowRuntimeId'])
")
```

---

### `createGate`

Creates a gate (entry or exit criteria) on a stage.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--gateType` | string | `PRE` (entry gate) or `POST` (exit gate) |
| `--condition` | string | Groovy condition expression |

**Examples:**

```bash
ectool createGate MyWebApp "CI-CD Pipeline" "Deploy-Production" \
  --gateType PRE \
  --condition 'true'
```

---

### `pausePipelineRun` / `resumePipelineRun` / `restartPipelineRun`

```bash
# Pause
ectool pausePipelineRun --flowRuntimeId "$FLOW_ID"

# Resume
ectool resumePipelineRun --flowRuntimeId "$FLOW_ID"

# Restart from a stage
NEW_FLOW_ID=$(ectool restartPipelineRun \
  --flowRuntimeId "$FLOW_ID" \
  --startingStage "Deploy-Staging" \
  --format json | python3 -c "
import sys, json
print(json.load(sys.stdin)['flowRuntime']['flowRuntimeId'])
")
```

---

### `retryTask`

Retries a failed task in an active pipeline run.

```bash
ectool retryTask \
  --flowRuntimeId "$FLOW_ID" \
  --stageName Test \
  --taskName "Integration Tests"
```

---

### `getAllWaitingTasks`

Lists all tasks across all active pipeline runs waiting for manual action.

```bash
ectool getAllWaitingTasks --format json | python3 -c "
import sys, json
tasks = json.load(sys.stdin).get('task', [])
print(f'Waiting tasks: {len(tasks)}')
for t in tasks:
    print(f\"  {t.get('taskName')} in {t.get('stageName')} ({t.get('flowRuntimeId')})\"
)"
```

---

### `getPipelineRunAuditReport`

Generates an audit report for a pipeline run.

```bash
# Save audit report for compliance
ectool getPipelineRunAuditReport \
  --flowRuntimeId "$FLOW_ID" \
  --format json > "/audit/pipeline-${FLOW_ID}-$(date +%Y%m%d).json"
```

---

### `deletePipelineRun`

Deletes completed pipeline run records for housekeeping.

```bash
# Delete all completed runs older than 60 days
ectool getPipelineRuns --format json | python3 -c "
import sys, json, subprocess
from datetime import datetime, timedelta, timezone
runs = json.load(sys.stdin).get('flowRuntime', [])
cutoff = datetime.now(timezone.utc) - timedelta(days=60)
for r in runs:
    if r.get('status') in ('completed', 'aborted', 'failed'):
        ts = r.get('createTime', '')
        try:
            created = datetime.fromisoformat(ts.replace('Z', '+00:00'))
            if created < cutoff:
                subprocess.run(['ectool', 'deletePipelineRun',
                  '--flowRuntimeId', r['flowRuntimeId']])
                print(f\"Deleted: {r['flowRuntimeId']}\")
        except: pass
"
```

---

### `modifyGate`

Modifies a gate condition on a pipeline stage.

```bash
ectool modifyGate \
  --projectName MyWebApp \
  --pipelineName "CI-CD Pipeline" \
  --stageName "Deploy-Production" \
  --gateType PRE \
  --condition 'property("/myJob/testCoverage").toInteger() >= 80' \
  --description "Require 80% test coverage before production deploy"
```

---

## Shell Scripting Patterns

### Full pipeline scaffold

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT="MyWebApp"
PIPELINE="CI-CD Pipeline"
RESOURCE="agent-linux-01"

ectool createPipeline "$PROJECT" "$PIPELINE" --resourceName "$RESOURCE"

for STAGE in Build Test "Deploy-Staging" "Deploy-Production"; do
    ectool createStage "$PROJECT" "$PIPELINE" "$STAGE"
done

ectool createTask "$PROJECT" "$PIPELINE" Build "Maven Build" \
  --taskType PROCEDURE --subproject "$PROJECT" --subprocedure Build \
  --actualParameter branch=main

ectool createTask "$PROJECT" "$PIPELINE" Test "Integration Tests" \
  --taskType PROCEDURE --subproject "$PROJECT" --subprocedure IntegrationTest

ectool createTask "$PROJECT" "$PIPELINE" "Deploy-Production" "Approve" \
  --taskType MANUAL \
  --instruction "Review staging results before production deployment."

echo "Pipeline '$PIPELINE' created in project '$PROJECT'."
```

### Run pipeline and wait with polling

```bash
#!/usr/bin/env bash
FLOW_ID=$(ectool runPipeline MyWebApp "CI-CD Pipeline" \
  --actualParameter branch=main \
  --format json | python3 -c "
import sys, json; print(json.load(sys.stdin)['flowRuntime']['flowRuntimeId'])")

echo "Flow runtime: $FLOW_ID"
while true; do
    STATUS=$(ectool getPipelineRun "$FLOW_ID" --format json | \
      python3 -c "import sys,json; print(json.load(sys.stdin)['flowRuntime']['status'])")
    echo "[$(date +%H:%M:%S)] Status: $STATUS"
    [[ "$STATUS" =~ ^(completed|failed|aborted)$ ]] && break
    sleep 20
done

OUTCOME=$(ectool getPipelineRun "$FLOW_ID" --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['flowRuntime'].get('outcome','unknown'))")
echo "Pipeline outcome: $OUTCOME"
[[ "$OUTCOME" == "success" ]] || exit 1
```
