# ectool — Job & Property

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
| `getJob` | Get job details and status | `jobId` |
| `getJobs` | List jobs (with filter) | _(none)_ |
| `abortJob` | Abort a running job | `jobId` |
| `deleteJob` | Delete a completed job | `jobId` |
| `waitForJob` | Block until job completes | `jobId` |
| `getJobStatus` | Get brief job status | `jobId` |
| `getJobStep` | Get details of a single step | `jobStepId` |
| `getJobSteps` | List all steps in a job | `jobId` |
| `getJobStepLog` | Retrieve step log output | `jobStepId` |
| `setProperty` | Set a property value | `propertyName` `value` |
| `getProperty` | Get a property value | `propertyName` |
| `deleteProperty` | Delete a property | `propertyName` |
| `getProperties` | List properties at a path | `propertyName` |
| `createProperty` | Create a new property | `propertyName` |
| `expandString` | Expand a string with property references | `value` |
| `createJobStep` | Dynamically create a step inside a running job | _(none — all named)_ |
| `setOutputParameter` | Set output parameter on running job step | `name` `value` |
| `getOutputParameters` | Get output parameters from a job | `jobId` |
| `abortAllJobs` | Abort all running jobs server-wide | _(none)_ |
| `abortJobStep` | Abort a single running job step | `jobStepId` |
| `completeJobStep` | Mark a job step as completed with status | _(none — all named)_ |
| `getJobDetails` | Full job data including all step details | `jobId` |
| `getJobInfo` | Lightweight job info (status, outcome, timing) | `jobId` |
| `getJobSummary` | Summary of job results | `jobId` |
| `incrementProperty` | Atomically increment an integer property | `propertyName` `incrementBy` |
| `modifyProperty` | Modify a property's metadata | `propertyName` |

---

### `getJob`

Retrieves full details of a job including status, outcome, steps, and parameters.

**Examples:**

```bash
# Get job as JSON and extract outcome
ectool getJob job-12345-abcde --format json | python3 -c "
import sys, json
job = json.load(sys.stdin)['job']
print('Status:', job['status'])
print('Outcome:', job.get('outcome', 'pending'))
"
```

---

### `waitForJob`

Blocks execution until the specified job finishes.

**Examples:**

```bash
# Wait with 10-minute timeout
ectool waitForJob "$JOB_ID" --timeout 600

# Full run-and-wait pattern
JOB_ID=$(ectool runProcedure MyWebApp Build \
  --actualParameter branch=main \
  --format json | python3 -c "import sys,json; print(json.load(sys.stdin)['job']['jobId'])")
ectool waitForJob "$JOB_ID" --timeout 1800 --pollInterval 10
```

---

### `getJobStepLog`

Retrieves the console output (log) of a specific job step.

**Examples:**

```bash
# Get all logs for a step
ectool getJobStepLog jobstep-12345-abcde

# Get logs for all steps of a job
STEPS=$(ectool getJobSteps "$JOB_ID" --format json | python3 -c "
import sys, json
steps = json.load(sys.stdin).get('jobStep', [])
for s in steps:
    print(s['jobStepId'], s.get('stepName', 'unnamed'))
")
while IFS=' ' read -r STEP_ID STEP_NAME; do
    echo "=== Step: $STEP_NAME ==="
    ectool getJobStepLog "$STEP_ID" 2>/dev/null
done <<< "$STEPS"
```

---

### `setProperty`

Sets a property value. Properties can be on jobs, steps, resources, projects, or the server.

**Examples:**

```bash
# Set a property on the current job (from inside a step)
ectool setProperty /myJob/build_version "2.5.1"

# Set a project-level property
ectool setProperty /myProject/releaseChannel "stable" \
  --projectName MyWebApp

# Set on a specific job
ectool setProperty /myJob/status "DEPLOYED" --jobId "$JOB_ID"
```

---

### `getProperty`

Retrieves a property value.

**Examples:**

```bash
# Get a project property value
VALUE=$(ectool getProperty /myProject/releaseChannel \
  --projectName MyWebApp --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['property']['value'])")
echo "Release channel: $VALUE"
```

---

### `createJobStep`

Dynamically creates a new step inside an already-running job.

**Examples:**

```bash
# Create a step that calls a subprocedure with parameters
ectool createJobStep \
  --jobStepId 5da765dd-73f1-11e3-b67e-b0a420524153 \
  --projectName Default \
  --subprocedure Deploy \
  --actualParameter env=production \
  --actualParameter version=2.5.1 \
  --errorHandling abortJob

# Create two parallel steps dynamically
ectool createJobStep \
  --jobStepId "$PARENT_STEP_ID" \
  --parallel 1 \
  --projectName Default \
  --subprocedure RunTests \
  --actualParameter suite=unit

ectool createJobStep \
  --jobStepId "$PARENT_STEP_ID" \
  --parallel 1 \
  --projectName Default \
  --subprocedure RunTests \
  --actualParameter suite=integration
```

---

### `incrementProperty`

Atomically increments an integer property. Thread-safe — safe to call from multiple concurrent job steps.

**Examples:**

```bash
# Increment a job-level counter from a parallel step
ectool incrementProperty /myJob/passedTests 1 --jobId "$JOB_ID"

# Increment a project-level build counter
ectool incrementProperty /myProject/buildCount 1 \
  --projectName MyWebApp
```

---

## Shell Scripting Patterns

### Wait for job and print step logs on failure

```bash
#!/usr/bin/env bash
set -euo pipefail

JOB_ID=$(ectool runProcedure MyWebApp Build \
  --actualParameter branch="${BRANCH:-main}" \
  --format json | python3 -c "import sys,json; print(json.load(sys.stdin)['job']['jobId'])")

echo "Job started: $JOB_ID"
ectool waitForJob "$JOB_ID" --timeout 900

OUTCOME=$(ectool getJob "$JOB_ID" --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['job']['outcome'])")

if [[ "$OUTCOME" != "success" ]]; then
    echo "Job failed (outcome=$OUTCOME). Printing step logs..."
    ectool getJobSteps "$JOB_ID" --format json | python3 -c "
import sys, json, subprocess
steps = json.load(sys.stdin).get('jobStep', [])
for s in steps:
    if s.get('outcome') in ('error', 'warning'):
        print('--- Step:', s.get('stepName'), '---')
        subprocess.run(['ectool', 'getJobStepLog', s['jobStepId']])
"
    exit 1
fi

echo "Build succeeded!"
```
