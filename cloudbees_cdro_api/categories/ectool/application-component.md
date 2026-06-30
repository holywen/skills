# ectool — Application & Component

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
| `createApplication` | Create an application | `projectName` `applicationName` |
| `modifyApplication` | Modify an application | `projectName` `applicationName` |
| `deleteApplication` | Delete an application | `projectName` `applicationName` |
| `getApplication` | Get application details | `projectName` `applicationName` |
| `getApplications` | List applications in a project | `projectName` |
| `deployApplication` | Deploy an application to an environment | `projectName` `applicationName` |
| `createComponent` | Create a component in an application | `projectName` `applicationName` `componentName` |
| `modifyComponent` | Modify a component | `projectName` `applicationName` `componentName` |
| `deleteComponent` | Delete a component | `projectName` `applicationName` `componentName` |
| `getComponent` | Get component details | `projectName` `applicationName` `componentName` |
| `getComponents` | List components in an application | `projectName` `applicationName` |
| `copyComponent` | Copy a component to a new name or project | _(none)_ |
| `createApplicationTier` | Create an application tier | _(none)_ |
| `deleteApplicationTier` | Delete an application tier | _(none)_ |
| `getApplicationTier` | Get an application tier | _(none)_ |
| `getApplicationTiers` | List all tiers in an application | _(none)_ |
| `modifyApplicationTier` | Modify an application tier | _(none)_ |
| `addComponentToApplicationTier` | Add a component to an application tier | _(none)_ |
| `removeComponentFromApplicationTier` | Remove a component from a tier | _(none)_ |
| `getComponentsInApplicationTier` | List components in a tier | _(none)_ |
| `createProcess` | Create a process on application or component | `projectName` `applicationName` `processName` |
| `modifyProcess` | Modify a process | `projectName` `applicationName` `processName` |
| `deleteProcess` | Delete a process | `projectName` `applicationName` `processName` |
| `getProcess` | Get process details | `projectName` `applicationName` `processName` |
| `getProcesses` | List all processes for an application or component | _(none)_ |
| `createProcessStep` | Create a step in a process | `projectName` `applicationName` `processName` `processStepName` |
| `modifyProcessStep` | Modify a process step | `projectName` `applicationName` `processName` `processStepName` |
| `deleteProcessStep` | Delete a process step | `projectName` `applicationName` `processName` `processStepName` |
| `getProcessStep` | Get a single process step | _(none)_ |
| `getProcessSteps` | List all steps in a process | _(none)_ |
| `createProcessDependency` | Create an ordering dependency between process steps | _(none)_ |
| `deleteProcessDependency` | Delete a process step dependency | _(none)_ |
| `getProcessDependencies` | List all dependencies in a process | _(none)_ |
| `modifyProcessDependency` | Modify a dependency | _(none)_ |
| `retryProcessStep` | Retry a failed process step in a running deployment | _(none)_ |
| `completeManualProcessStep` | Complete a manual process step in a running deployment | _(none)_ |
| `runProcess` | Run an application or component process | `projectName` `applicationName` |
| `createDeployerApplication` | Create a deployer application config | _(none)_ |
| `modifyDeployerApplication` | Modify a deployer application | _(none)_ |
| `deleteDeployerApplication` | Delete a deployer application | _(none)_ |
| `getDeployerApplication` | Get a deployer application config | _(none)_ |
| `getDeployerApplications` | List deployer applications | _(none)_ |
| `getDeployerConfiguration` | Get a deployer configuration | _(none)_ |
| `getDeployerConfigurations` | List deployer configurations | _(none)_ |
| `modifyDeployerConfiguration` | Modify a deployer configuration | _(none)_ |
| `removeDeployerApplication` | Remove a deployer application from a task | _(none)_ |
| `validateDeployer` | Validate a deployer task configuration | _(none)_ |
| `setTierResourcePhase` | Set the phase of a resource in a tier for rolling deploy | _(none)_ |
| `getDeploymentHistoryItems` | Get deployment history for an environment | _(none)_ |

---

### `createApplication`

Creates an application definition within a project.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Owning project |
| `applicationName` | Yes | Unique application name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Human-readable description |
| `--resourceName` | string | Default resource for processes |
| `--workspaceName` | string | Default workspace |
| `--credentialName` | string | Default credential |

**Examples:**

```bash
# Create application
ectool createApplication MyProject "MyWebApp" \
  --description "Customer-facing web application" \
  --resourceName agent-linux-01

# List applications as JSON
ectool getApplications MyProject --format json | python3 -c "
import sys, json
apps = json.load(sys.stdin).get('application', [])
for a in apps:
    print(a['applicationName'])
"
```

---

### `createComponent`

Creates a component (deployable artifact unit) within an application.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `applicationName` | Yes | Parent application |
| `componentName` | Yes | Component name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Description |
| `--resourceName` | string | Default resource |
| `--workspaceName` | string | Default workspace |
| `--pluginKey` | string | Plugin that manages this component (e.g. `EC-Artifact`) |
| `--pluginConfigurationName` | string | Plugin configuration name |
| `--reference` | boolean | Mark as a reference component |

**Examples:**

```bash
# Create a WAR file component
ectool createComponent MyProject MyWebApp "webapp-war" \
  --description "Web application WAR file" \
  --resourceName agent-linux-01

# Create a database schema component
ectool createComponent MyProject MyWebApp "db-schema" \
  --description "Database migration scripts" \
  --resourceName db-agent-01

# Create a configuration component
ectool createComponent MyProject MyWebApp "app-config" \
  --description "Application configuration files"
```

---

### `copyComponent`

Copies an existing component to a new name within the same application or to a different application.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the source application |
| `--applicationName` | string | Source application containing the component |
| `--componentName` | string | Name of the component to copy |
| `--cloneName` | string | Name for the new (copied) component |
| `--cloneApplicationName` | string | Destination application (if copying to a different application) |

**Examples:**

```bash
# Copy a component within the same application
ectool copyComponent \
  --projectName MyProject \
  --applicationName MyWebApp \
  --componentName webapp-war \
  --cloneName webapp-war-v2

# Copy a component to a different application
ectool copyComponent \
  --projectName MyProject \
  --applicationName MyWebApp \
  --componentName db-schema \
  --cloneName db-schema \
  --cloneApplicationName MyWebApp-EU
```

---

### `createApplicationTier`

Creates an application tier, which is a logical grouping within an application that maps to a set of environment tier resources.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the application |
| `--applicationName` | string | Application to add the tier to |
| `--applicationTierName` | string | Name of the new tier |

**Examples:**

```bash
# Create a web tier for an application
ectool createApplicationTier \
  --projectName MyProject \
  --applicationName MyWebApp \
  --applicationTierName web-tier

# Create all standard tiers for an application
for TIER in web-tier app-tier db-tier cache-tier; do
  ectool createApplicationTier \
    --projectName MyProject \
    --applicationName MyWebApp \
    --applicationTierName "$TIER"
  echo "Created tier: $TIER"
done
```

---

### `addComponentToApplicationTier`

Assigns a component to an application tier.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the application |
| `--applicationName` | string | Application containing the tier |
| `--applicationTierName` | string | Tier to add the component to |
| `--componentName` | string | Component to assign to the tier |

**Examples:**

```bash
ectool addComponentToApplicationTier \
  --projectName MyProject \
  --applicationName MyWebApp \
  --applicationTierName web-tier \
  --componentName webapp-war

ectool addComponentToApplicationTier \
  --projectName MyProject \
  --applicationName MyWebApp \
  --applicationTierName db-tier \
  --componentName db-schema
```

---

### `createProcess`

Creates a process (deployment workflow) on an application or component.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `applicationName` | Yes | Application name |
| `processName` | Yes | Process name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--description` | string | Process description |
| `--processType` | string | `DEPLOY`, `UNDEPLOY`, `ROLLBACK`, `OTHER` |
| `--componentName` | string | Attach process to a component (instead of application) |
| `--resourceName` | string | Default resource |

**Examples:**

```bash
# Application-level deploy process
ectool createProcess MyProject MyWebApp Deploy \
  --processType DEPLOY \
  --description "Full application deployment"

# Component-level deploy process
ectool createProcess MyProject MyWebApp "Deploy WAR" \
  --processType DEPLOY \
  --componentName webapp-war

# Rollback process
ectool createProcess MyProject MyWebApp Rollback \
  --processType ROLLBACK
```

---

### `createProcessStep`

Creates a step within a process. Steps can call plugins, subprocedures, or run commands.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `applicationName` | Yes | Application name |
| `processName` | Yes | Process name |
| `processStepName` | Yes | Step name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--componentName` | string | Component this step belongs to (if component process) |
| `--processStepType` | string | `command`, `procedure`, `plugin`, `component`, `manual` |
| `--subprocedure` | string | Procedure to call |
| `--subproject` | string | Project for subprocedure |
| `--subcomponentProcess` | string | Component process to call |
| `--subcomponent` | string | Component for subcomponentProcess |
| `--command` | string | Shell command |
| `--shell` | string | Shell interpreter |
| `--actualParameter` | string (repeatable) | `name=value` parameters |
| `--errorHandling` | string | `failProcess`, `abortJob`, `ignore` |

**Examples:**

```bash
# Step that calls a component process
ectool createProcessStep MyProject MyWebApp Deploy "Deploy WAR" \
  --processStepType component \
  --subcomponent webapp-war \
  --subcomponentProcess "Deploy WAR"

# Step running a shell command
ectool createProcessStep MyProject MyWebApp Deploy "Health Check" \
  --processStepType command \
  --shell bash \
  --command 'curl -sf http://localhost:8080/health || exit 1'

# Step calling a subprocedure
ectool createProcessStep MyProject MyWebApp Deploy "Run DB Migration" \
  --processStepType procedure \
  --subproject MyProject \
  --subprocedure RunFlywayMigration
```

---

### `createProcessDependency`

Creates an ordering dependency between two process steps.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project name |
| `--applicationName` | string | Application name |
| `--processName` | string | Process containing the steps |
| `--processStepName` | string | The downstream step (the step that waits) |
| `--dependsOn` | string | The upstream step (the step that must complete first) |

**Examples:**

```bash
ectool createProcessDependency \
  --projectName MyProject \
  --applicationName MyWebApp \
  --processName Deploy \
  --processStepName "Deploy WAR" \
  --dependsOn "Stop Tomcat"

ectool createProcessDependency \
  --projectName MyProject \
  --applicationName MyWebApp \
  --processName Deploy \
  --processStepName "Health Check" \
  --dependsOn "Start App"
```

---

### `deployApplication`

Deploys an application to a specified environment using its Deploy process.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project name |
| `applicationName` | Yes | Application to deploy |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--environmentName` | string | Target environment |
| `--environmentProjectName` | string | Project owning the environment |
| `--processName` | string | Process to run (default: `Deploy`) |
| `--actualParameter` | string (repeatable) | `name=value` parameters |
| `--snapshotName` | string | Deploy from a snapshot |

**Examples:**

```bash
# Deploy to staging
ectool deployApplication MyProject MyWebApp \
  --environmentName staging \
  --environmentProjectName MyProject \
  --actualParameter version=2.5.1

# Deploy and capture job ID
JOB_ID=$(ectool deployApplication MyProject MyWebApp \
  --environmentName staging \
  --environmentProjectName MyProject \
  --format json | python3 -c "
import sys, json
print(json.load(sys.stdin)['job']['jobId'])
")
ectool waitForJob "$JOB_ID" --timeout 600
```

---

### `validateDeployer`

Validates the configuration of a deployer task.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the pipeline |
| `--pipelineName` | string | Pipeline containing the deployer task |
| `--stageName` | string | Stage containing the task |
| `--taskName` | string | Deployer task to validate |

**Examples:**

```bash
ectool validateDeployer \
  --projectName MyProject \
  --pipelineName "CI-CD Pipeline" \
  --stageName "Deploy to Staging" \
  --taskName "Deploy Applications"
echo "Validation passed — triggering release pipeline"
ectool runPipeline MyProject "Release Pipeline"
```

---

### `getDeploymentHistoryItems`

Gets the deployment history records for an environment.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the environment |
| `--environmentName` | string | Environment to query |

**Examples:**

```bash
ectool getDeploymentHistoryItems \
  --projectName MyProject \
  --environmentName production \
  --format json | python3 -c "
import sys, json
items = json.load(sys.stdin).get('deploymentHistoryItem', [])
for item in items:
    print(f\"{item.get('startTime','')[:19]}  {item.get('applicationName',''):20s}  {item.get('status','')}\")
"
```

---

## Shell Scripting Patterns

### Full application scaffold

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT="MyProject"
APP="MyWebApp"
RESOURCE="agent-linux-01"

ectool createApplication "$PROJECT" "$APP" --resourceName "$RESOURCE"

for COMP in webapp-war db-schema app-config; do
    ectool createComponent "$PROJECT" "$APP" "$COMP"
done

for TIER in web-tier app-tier db-tier; do
    ectool createApplicationTier \
      --projectName "$PROJECT" \
      --applicationName "$APP" \
      --applicationTierName "$TIER"
done

ectool addComponentToApplicationTier \
  --projectName "$PROJECT" --applicationName "$APP" \
  --applicationTierName web-tier --componentName webapp-war
ectool addComponentToApplicationTier \
  --projectName "$PROJECT" --applicationName "$APP" \
  --applicationTierName db-tier --componentName db-schema
ectool addComponentToApplicationTier \
  --projectName "$PROJECT" --applicationName "$APP" \
  --applicationTierName app-tier --componentName app-config

ectool createProcess "$PROJECT" "$APP" Deploy --processType DEPLOY

ectool createProcessStep "$PROJECT" "$APP" Deploy "Stop App" \
  --processStepType command --shell bash \
  --command 'systemctl stop mywebapp || true'

ectool createProcessStep "$PROJECT" "$APP" Deploy "Deploy WAR" \
  --processStepType component \
  --subcomponent webapp-war \
  --subcomponentProcess "Deploy WAR"

ectool createProcessStep "$PROJECT" "$APP" Deploy "Start App" \
  --processStepType command --shell bash \
  --command 'systemctl start mywebapp'

ectool createProcessStep "$PROJECT" "$APP" Deploy "Health Check" \
  --processStepType command --shell bash \
  --command 'curl -sf http://localhost:8080/health || exit 1'

ectool createProcessDependency \
  --projectName "$PROJECT" --applicationName "$APP" \
  --processName Deploy \
  --processStepName "Deploy WAR" --dependsOn "Stop App"
ectool createProcessDependency \
  --projectName "$PROJECT" --applicationName "$APP" \
  --processName Deploy \
  --processStepName "Start App" --dependsOn "Deploy WAR"
ectool createProcessDependency \
  --projectName "$PROJECT" --applicationName "$APP" \
  --processName Deploy \
  --processStepName "Health Check" --dependsOn "Start App"

echo "Application '$APP' scaffolded in '$PROJECT'."
```

### Deploy and verify

```bash
#!/usr/bin/env bash
VERSION="${1:-latest}"
ENV="${2:-staging}"

JOB_ID=$(ectool deployApplication MyProject MyWebApp \
  --environmentName "$ENV" \
  --environmentProjectName MyProject \
  --actualParameter version="$VERSION" \
  --format json | python3 -c \
  "import sys,json; print(json.load(sys.stdin)['job']['jobId'])")

echo "Deploying version $VERSION to $ENV — job: $JOB_ID"
ectool waitForJob "$JOB_ID" --timeout 900

OUTCOME=$(ectool getJob "$JOB_ID" --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['job']['outcome'])")

if [[ "$OUTCOME" != "success" ]]; then
    echo "Deployment failed! Check job $JOB_ID for details."
    exit 1
fi

echo "Deployment of $VERSION to $ENV succeeded."
```
