# ectool — Project, Procedure & Step

Fetch docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

---

## Login / Session Setup

```bash
# Interactive login — stores session in ~/.ec/session
ectool login admin changeme --server cdro.example.com

# Environment-variable based (no interactive login needed)
export ELECTRIC_COMMANDER_SERVER=cdro.example.com
export COMMANDER_SESSIONID=<token>

# Verify connectivity
ectool getServerStatus
```

---

## Command Reference Table

| Command | Description | Positional Args (in order) |
|---------|-------------|---------------------------|
| `createProject` | Create a new project | `projectName` |
| `modifyProject` | Modify an existing project | `projectName` |
| `deleteProject` | Delete a project | `projectName` |
| `getProject` | Get project details | `projectName` |
| `getProjects` | List all projects | _(none)_ |
| `createProcedure` | Create a procedure in a project | `projectName` `procedureName` |
| `modifyProcedure` | Modify a procedure | `projectName` `procedureName` |
| `deleteProcedure` | Delete a procedure | `projectName` `procedureName` |
| `getProcedure` | Get procedure details | `projectName` `procedureName` |
| `getProcedures` | List procedures in a project | `projectName` |
| `runProcedure` | Run a procedure and return jobId | `projectName` `procedureName` |
| `createStep` | Create a step in a procedure | `projectName` `procedureName` `stepName` |
| `modifyStep` | Modify a step | `projectName` `procedureName` `stepName` |
| `deleteStep` | Delete a step | `projectName` `procedureName` `stepName` |
| `getStep` | Get step details | `projectName` `procedureName` `stepName` |
| `getSteps` | List all steps in a procedure | `projectName` `procedureName` |
| `copyProcedure` | Copy a procedure to another project | `projectName` `procedureName` |
| `moveProcedure` | Move a procedure to another project | `projectName` `procedureName` |

---

### `createProject`

Creates a new top-level project.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Unique name for the project |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Human-readable description |
| `--resourceName` | string | Default resource for jobs in this project |
| `--workspaceName` | string | Default workspace for jobs |
| `--credentialName` | string | Default credential |
| `--tracked` | boolean | Track artifact usage (1/0) |

**Examples:**

```bash
# Basic project creation
ectool createProject MyWebApp

# With description and default resource
ectool createProject MyWebApp \
  --description "Web application CI/CD pipeline" \
  --resourceName agent-linux-01 \
  --workspaceName /workspace/mywebapp

# List all projects as JSON
ectool getProjects --format json | python3 -c "
import sys, json
projects = json.load(sys.stdin)['project']
for p in projects:
    print(p['projectName'])
"
```

---

### `createProcedure`

Creates a procedure (workflow definition) within a project.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project that owns the procedure |
| `procedureName` | Yes | Name of the procedure |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Human-readable description |
| `--defaultResourceName` | string | Default agent resource for steps |
| `--defaultWorkspaceName` | string | Default workspace |
| `--jobNameTemplate` | string | Template for job name (e.g. `$[/myJob/projectName]-$[/myJob/jobId]`) |
| `--timeLimit` | string | Max runtime in minutes |
| `--timeLimitUnits` | string | `minutes`, `hours`, `seconds` |

**Examples:**

```bash
# Create a build procedure
ectool createProcedure MyWebApp Build \
  --description "Compile and unit test" \
  --defaultResourceName agent-linux-01 \
  --jobNameTemplate "Build-$[/myJob/jobId]"

# Create a deploy procedure with time limit
ectool createProcedure MyWebApp Deploy \
  --defaultResourceName agent-linux-01 \
  --timeLimit 30 \
  --timeLimitUnits minutes
```

---

### `createStep`

Creates a step within a procedure. Steps are the executable units.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `procedureName` | Yes | Procedure name |
| `stepName` | Yes | Step name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--command` | string | Shell command(s) to execute |
| `--shell` | string | Shell interpreter (`bash`, `sh`, `ec-perl`, `python`) |
| `--resourceName` | string | Override resource for this step |
| `--workspaceName` | string | Override workspace |
| `--condition` | string | Run condition expression |
| `--parallel` | boolean | Run in parallel with previous step |
| `--alwaysRun` | boolean | Run even if prior steps fail |
| `--timeLimit` | string | Max step runtime |
| `--timeLimitUnits` | string | `minutes`, `hours`, `seconds` |
| `--errorHandling` | string | `failProcedure` (default), `abortJob`, `ignore` |
| `--subprocedure` | string | Call another procedure as a step |
| `--subproject` | string | Project for subprocedure |
| `--subpluginKey` | string | Plugin key for plugin subprocedures |

**Examples:**

```bash
# Simple shell step
ectool createStep MyWebApp Build "Checkout" \
  --shell bash \
  --command "git clone https://github.com/org/mywebapp.git /workspace/src"

# Step calling Maven
ectool createStep MyWebApp Build "Maven Build" \
  --shell bash \
  --command "cd /workspace/src && mvn clean package -DskipTests=false" \
  --timeLimit 20 \
  --timeLimitUnits minutes \
  --errorHandling failProcedure

# Step that always runs (e.g. cleanup)
ectool createStep MyWebApp Build "Cleanup" \
  --shell bash \
  --command "rm -rf /workspace/src" \
  --alwaysRun 1

# Step calling a subprocedure
ectool createStep MyWebApp Deploy "Run Smoke Tests" \
  --subproject MyWebApp \
  --subprocedure SmokeTests

# Step with condition — only on main branch
ectool createStep MyWebApp Build "Publish Artifact" \
  --shell bash \
  --command "ectool publishArtifactVersion ..." \
  --condition '$[branch] eq "main"'
```

---

### `runProcedure`

Submits a procedure for execution. Returns a jobId.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project containing the procedure |
| `procedureName` | Yes | Procedure to run |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--actualParameter` | string (repeatable) | Pass `name=value` parameters (repeat flag for each) |
| `--resourceName` | string | Override default resource |
| `--workspaceName` | string | Override default workspace |
| `--priority` | string | `low`, `normal`, `high` |
| `--scheduleName` | string | Associate with a schedule |

**Examples:**

```bash
# Simple run
ectool runProcedure MyWebApp Build

# Run with parameters
ectool runProcedure MyWebApp Build \
  --actualParameter branch=main \
  --actualParameter version=2.5.1 \
  --actualParameter skipTests=false

# Capture jobId and wait for completion
JOB_ID=$(ectool runProcedure MyWebApp Build \
  --actualParameter branch=main \
  --format json | python3 -c "import sys,json; print(json.load(sys.stdin)['job']['jobId'])")

echo "Started job: $JOB_ID"
ectool waitForJob "$JOB_ID" --timeout 600

# Check result
STATUS=$(ectool getJob "$JOB_ID" --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['job']['outcome'])")
echo "Job outcome: $STATUS"
[ "$STATUS" = "success" ] || exit 1
```

---

### `copyProcedure`

Copies a procedure (and its steps) to another location.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Source project |
| `procedureName` | Yes | Source procedure |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--destinationProjectName` | string | Target project (defaults to same project) |
| `--destinationProcedureName` | string | Name for the copy |

**Examples:**

```bash
# Copy procedure within same project
ectool copyProcedure MyWebApp Build \
  --destinationProcedureName Build-v2

# Copy to another project
ectool copyProcedure MyWebApp Build \
  --destinationProjectName MyWebApp-Staging \
  --destinationProcedureName Build
```

---

## Shell Scripting Patterns

### Create full project scaffold

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT="MyWebApp"
RESOURCE="agent-linux-01"

# Create project
ectool createProject "$PROJECT" \
  --description "Main web application" \
  --resourceName "$RESOURCE"

# Create Build procedure
ectool createProcedure "$PROJECT" Build \
  --defaultResourceName "$RESOURCE" \
  --jobNameTemplate "Build-$[/myJob/jobId]"

# Add steps
ectool createStep "$PROJECT" Build Checkout \
  --shell bash \
  --command 'git clone $[gitUrl] /workspace/src'

ectool createStep "$PROJECT" Build Compile \
  --shell bash \
  --command 'cd /workspace/src && mvn clean package'

ectool createStep "$PROJECT" Build Test \
  --shell bash \
  --command 'cd /workspace/src && mvn test'

echo "Project scaffold created."
```

### Run and poll with JSON output

```bash
#!/usr/bin/env bash
JOB_JSON=$(ectool runProcedure MyWebApp Build \
  --actualParameter branch=main \
  --format json)

JOB_ID=$(echo "$JOB_JSON" | python3 -c \
  "import sys,json; print(json.load(sys.stdin)['job']['jobId'])")

echo "Launched job $JOB_ID — waiting..."
ectool waitForJob "$JOB_ID" --timeout 1800

OUTCOME=$(ectool getJob "$JOB_ID" --format json | python3 -c \
  "import sys,json; d=json.load(sys.stdin)['job']; print(d['outcome'])")

echo "Outcome: $OUTCOME"
[["$OUTCOME" == "success"]] || { echo "Job failed!"; exit 1; }
```
