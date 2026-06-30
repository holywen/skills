# ectool — Parameter, Plugin & Email

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
| `createFormalParameter` | Create a formal parameter on a procedure, pipeline, or process | `parameterName` |
| `modifyFormalParameter` | Modify a formal parameter | `parameterName` |
| `deleteFormalParameter` | Delete a formal parameter | `parameterName` |
| `getFormalParameter` | Get formal parameter details | `parameterName` |
| `getFormalParameters` | List formal parameters | _(varies)_ |
| `setOutputParameter` | Set an output parameter from within a running step | `name` `value` |
| `getOutputParameter` | Get a single output parameter | _(none)_ |
| `getOutputParameters` | Get output parameters from a completed job | `jobId` |
| `getActualParameter` | Get a single actual parameter | _(none)_ |
| `getActualParameters` | List all actual parameters from a container | _(varies)_ |
| `installPlugin` | Install a plugin from a file or URL | `filePath` |
| `uninstallPlugin` | Uninstall a plugin | `pluginName` |
| `promotePlugin` | Promote a plugin version to active | `pluginName` |
| `demotePlugin` | Demote (deactivate) a plugin | `pluginName` |
| `getPlugin` | Get plugin details | `pluginName` |
| `getPlugins` | List installed plugins | _(none)_ |
| `deletePlugin` | Delete a plugin metadata object | `pluginName` |
| `createPluginConfiguration` | Create a plugin configuration | `pluginName` `configName` |
| `modifyPluginConfiguration` | Modify a plugin configuration | `pluginName` `configName` |
| `deletePluginConfiguration` | Delete a plugin configuration | _(none)_ |
| `getPluginConfiguration` | Get a plugin configuration | _(none)_ |
| `getPluginConfigurations` | List plugin configurations | _(none)_ |
| `createEmailConfig` | Create an SMTP email configuration | `emailConfigName` |
| `modifyEmailConfig` | Modify email configuration | `emailConfigName` |
| `deleteEmailConfig` | Delete email configuration | `emailConfigName` |
| `getEmailConfig` | Get email configuration details | `emailConfigName` |
| `sendEmail` | Send an email notification | _(none)_ |

---

### `createFormalParameter`

Declares an input parameter on a procedure, pipeline, process, or schedule.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project context |
| `--procedureName` | string | Attach to a procedure |
| `--pipelineName` | string | Attach to a pipeline |
| `--type` | string | `text`, `textarea`, `password`, `checkbox`, `select`, `entry`, `credential` |
| `--defaultValue` | string | Default value |
| `--required` | boolean | Whether required (1/0) |
| `--description` | string | Parameter description shown in UI |
| `--optionValue` | string (repeatable) | Option values (for `select` type) |

**Examples:**

```bash
# Text parameter for branch name
ectool createFormalParameter branch \
  --projectName MyProject \
  --procedureName Build \
  --type text \
  --defaultValue main \
  --required 1 \
  --description "Git branch to build"

# Select parameter for environment
ectool createFormalParameter environment \
  --projectName MyProject \
  --procedureName Deploy \
  --type select \
  --defaultValue staging \
  --required 1 \
  --optionValue staging \
  --optionValue uat \
  --optionValue production

# Password parameter
ectool createFormalParameter dbPassword \
  --projectName MyProject \
  --procedureName Deploy \
  --type password \
  --required 1
```

---

### `setOutputParameter`

Sets an output parameter from within a running step.

**Examples:**

```bash
# Set output parameter from within a step command
ectool setOutputParameter artifact_version "2.5.1"
ectool setOutputParameter build_number "$(cat /workspace/build.number)"
ectool setOutputParameter docker_image "registry.example.com/myapp:2.5.1"

# Reference in downstream step:
# $[/myJob/jobSteps[StepName]/artifact_version]
```

---

### `installPlugin` and `promotePlugin`

**Examples:**

```bash
# Install a plugin from file
ectool installPlugin /tmp/EC-Git-2.5.0.jar

# Install and promote in one step
ectool installPlugin /tmp/EC-Docker-5.0.0.jar
ectool promotePlugin EC-Docker-5.0.0

# Check installed plugins
ectool getPlugins --format json | python3 -c "
import sys, json
plugins = json.load(sys.stdin).get('plugin', [])
for p in plugins:
    promoted = '(active)' if p.get('promoted') else ''
    print(f"{p['pluginName']:40s} {p.get('pluginVersion',''):10s} {promoted}")
"
```

---

### `createPluginConfiguration`

Creates a named configuration for a plugin.

**Examples:**

```bash
# Create a Git plugin configuration
ectool createPluginConfiguration EC-Git github-config \
  --description "GitHub corporate instance" \
  --parameter repoUrl=https://github.com/myorg \
  --credential github-credentials

# Create a Kubernetes configuration
ectool createPluginConfiguration EC-Kubernetes k8s-prod \
  --description "Production Kubernetes cluster" \
  --parameter apiServerUrl=https://k8s.prod.example.com:6443 \
  --credential k8s-service-account
```

---

### `createEmailConfig` and `sendEmail`

**Examples:**

```bash
# Create SMTP configuration
ectool createEmailConfig smtp-corporate \
  --mailServer smtp.example.com \
  --port 587 \
  --senderAddress cdro-noreply@example.com \
  --userName smtp-user \
  --password "SmtpP@ss!" \
  --ssl 1

# Send a notification email
ectool sendEmail \
  --emailConfigName smtp-corporate \
  --to ops-team@example.com \
  --cc dev-lead@example.com \
  --subject "Build $BUILD_NUM succeeded" \
  --body "Build $BUILD_NUM on branch main completed successfully."

# Send HTML email
ectool sendEmail \
  --emailConfigName smtp-corporate \
  --to ops-team@example.com \
  --subject "Deployment Complete" \
  --html 1 \
  --body "<h2>Deployment Successful</h2><p>Version <strong>$VERSION</strong> deployed.</p>"
```

---

## Shell Scripting Patterns

### Post-job notification pattern

```bash
#!/usr/bin/env bash
JOB_ID="$1"
ENV="$2"
VERSION="$3"

ectool waitForJob "$JOB_ID" --timeout 1800

OUTCOME=$(ectool getJob "$JOB_ID" --format json | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['job']['outcome'])")

if [[ "$OUTCOME" == "success" ]]; then
    SUBJECT="Deployment SUCCESS: $VERSION -> $ENV"
    BODY="Version $VERSION was successfully deployed to $ENV. Job: $JOB_ID"
else
    SUBJECT="Deployment FAILED: $VERSION -> $ENV"
    BODY="Deployment of version $VERSION to $ENV failed. Job: $JOB_ID."
fi

ectool sendEmail \
  --emailConfigName smtp-corporate \
  --to ops-team@example.com \
  --subject "$SUBJECT" \
  --body "$BODY"
```
