# ectool — Schedule & Artifact

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
| `createSchedule` | Create a schedule to run a procedure or pipeline | `projectName` `scheduleName` |
| `modifySchedule` | Modify a schedule | `projectName` `scheduleName` |
| `deleteSchedule` | Delete a schedule | `projectName` `scheduleName` |
| `getSchedule` | Get schedule details | `projectName` `scheduleName` |
| `getSchedules` | List schedules in a project | `projectName` |
| `runSchedule` | Trigger a schedule immediately | `projectName` `scheduleName` |
| `publishArtifactVersion` | Publish an artifact to the artifact repository | `groupId` `artifactKey` `version` |
| `retrieveArtifactVersions` | Retrieve artifact versions to a local path | `groupId` `artifactKey` |
| `getArtifactVersion` | Get artifact version metadata | `groupId` `artifactKey` `version` |
| `getArtifactVersions` | List artifact versions | `groupId` `artifactKey` |
| `deleteArtifactVersion` | Delete an artifact version | `groupId` `artifactKey` `version` |
| `createArtifact` | Create an artifact definition | `groupId` `artifactKey` |
| `deleteArtifact` | Delete an artifact definition and all its versions | `groupId` `artifactKey` |
| `getArtifacts` | List artifacts | _(none)_ |
| `createSnapshot` | Create a snapshot of application component versions | `projectName` `snapshotName` |
| `deleteSnapshot` | Delete a snapshot | `projectName` `snapshotName` |
| `getSnapshot` | Get a snapshot definition | `projectName` `snapshotName` |
| `getSnapshots` | List all snapshots for an application | _(none)_ |
| `createTag` | Create a global tag for tagging objects | `tagName` |
| `deleteTag` | Delete a tag | `tagName` |
| `getTags` | List all defined tags | _(none)_ |
| `tagObject` | Attach a tag to a CD/RO object | _(none)_ |
| `untagObject` | Remove a tag from an object | _(none)_ |
| `cleanupRepository` | Purge old artifact versions from a repository | _(none)_ |

---

### `createSchedule`

Creates a schedule to run a procedure or pipeline on a cron expression or interval.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--procedureName` | string | Procedure to run |
| `--pipelineName` | string | Pipeline to run |
| `--cronExpression` | string | Cron schedule (e.g. `0 2 * * *` for 2 AM daily) |
| `--interval` | string | Interval in minutes |
| `--intervalUnits` | string | `minutes`, `hours`, `days` |
| `--timeZone` | string | IANA timezone |
| `--scheduleDisabled` | boolean | Create in disabled state (1/0) |
| `--actualParameter` | string (repeatable) | `name=value` parameters for the run |

**Examples:**

```bash
# Nightly build at 2 AM UTC
ectool createSchedule MyProject "Nightly Build" \
  --procedureName Build \
  --cronExpression "0 2 * * *" \
  --timeZone UTC \
  --actualParameter branch=main

# Every 30 minutes CI run
ectool createSchedule MyProject "CI Trigger" \
  --procedureName Build \
  --interval 30 \
  --intervalUnits minutes \
  --actualParameter branch=develop

# Weekly pipeline run (Sundays at midnight)
ectool createSchedule MyProject "Weekly Release" \
  --pipelineName "CI-CD Pipeline" \
  --cronExpression "0 0 * * 0" \
  --timeZone "America/New_York"
```

---

### `publishArtifactVersion`

Publishes artifact files to the CD/RO artifact repository.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--fromDirectory` | string | Local directory to publish from |
| `--includePatterns` | string | Glob pattern for files to include |
| `--excludePatterns` | string | Glob pattern for files to exclude |
| `--repositoryName` | string | Target repository |
| `--description` | string | Version description |

**Examples:**

```bash
# Publish a WAR file
ectool publishArtifactVersion com.example mywebapp 2.5.1 \
  --fromDirectory /workspace/src/target \
  --includePatterns "*.war" \
  --description "Build from branch main"

# Publish with build metadata
VERSION="$(git describe --tags --always)"
ectool publishArtifactVersion com.example mywebapp "$VERSION" \
  --fromDirectory /workspace/target \
  --includePatterns "*.war"
```

---

### `retrieveArtifactVersions`

Downloads artifact version files to a local directory.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--version` | string | Specific version (omit for latest) |
| `--toDirectory` | string | Local directory to download into |
| `--overwrite` | boolean | Overwrite existing files (1/0) |

**Examples:**

```bash
# Retrieve latest version to /opt/deploy
ectool retrieveArtifactVersions com.example mywebapp \
  --toDirectory /opt/deploy \
  --overwrite 1

# Retrieve specific version
ectool retrieveArtifactVersions com.example mywebapp \
  --version 2.5.1 \
  --toDirectory /opt/deploy/mywebapp-2.5.1
```

---

### `createSnapshot`

Creates a snapshot of an application, recording the current component versions.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--applicationName` | string | Application to snapshot |
| `--description` | string | Description |

**Examples:**

```bash
# Create a release candidate snapshot
ectool createSnapshot MyProject "v2.5.1-RC1" \
  --applicationName MyWebApp \
  --description "Release candidate 1 for version 2.5.1"

# Create snapshot and use it for production deploy
ectool createSnapshot MyProject "prod-deploy-v3" \
  --applicationName MyWebApp
ectool deployApplication MyProject MyWebApp \
  --environmentName production \
  --environmentProjectName MyProject \
  --snapshotName "prod-deploy-v3"
```

---

### `createTag` and `tagObject`

**Examples:**

```bash
# Create a tag
ectool createTag "production-ready" \
  --description "Marks objects approved for production deployment" \
  --color "#00AA00"

# Tag a pipeline
ectool tagObject \
  --tagName "production-ready" \
  --projectName MyProject \
  --pipelineName "CI-CD Pipeline"

# Remove a tag
ectool untagObject \
  --tagName "deprecated" \
  --projectName MyProject \
  --pipelineName "Old-Release-Pipeline"
```

---

## Shell Scripting Patterns

### Publish artifact in a build step

```bash
#!/usr/bin/env bash
set -euo pipefail

VERSION="${1:-$(date +%Y%m%d.%H%M%S)}"
GROUP="com.example"
ARTIFACT="mywebapp"
TARGET_DIR="/workspace/src/target"

echo "Building..."
cd /workspace/src && mvn clean package -DskipTests

echo "Publishing artifact $GROUP:$ARTIFACT:$VERSION..."
ectool publishArtifactVersion "$GROUP" "$ARTIFACT" "$VERSION" \
  --fromDirectory "$TARGET_DIR" \
  --includePatterns "*.war"

ectool setProperty /myJob/artifact_version "$VERSION"
echo "Published version: $VERSION"
```

### Manage schedules by enabling/disabling

```bash
#!/usr/bin/env bash
PROJECT="MyProject"
ACTION="${1:-enable}"

DISABLED=$([[ "$ACTION" == "disable" ]] && echo 1 || echo 0)

for SCHEDULE in "Nightly Build" "CI Trigger" "Weekly Release"; do
    ectool modifySchedule "$PROJECT" "$SCHEDULE" \
      --scheduleDisabled "$DISABLED"
    echo "$ACTION: $SCHEDULE"
done
```
