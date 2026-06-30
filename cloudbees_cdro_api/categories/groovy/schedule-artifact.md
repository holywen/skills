# CloudBees CD/RO Groovy API — Schedule & Artifact

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`
> Example: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/createSchedule`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow

// Option 1: environment variable
// export COMMANDER_SERVER=cdro.example.com

// Option 2: system property at runtime
// ec-groovy -DCOMMANDER_SERVER=cdro.example.com -DCOMMANDER_SECURE=1 script.groovy

ElectricFlow ef = new ElectricFlow()
```

---

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createSchedule` | Creates a cron-based or interval schedule | `projectName`, `scheduleName` |
| `deleteSchedule` | Deletes a schedule | `projectName`, `scheduleName` |
| `getSchedule` | Returns schedule details | `projectName`, `scheduleName` |
| `modifySchedule` | Updates schedule attributes | `projectName`, `scheduleName` |
| `runSchedule` | Manually triggers a schedule immediately | `projectName`, `scheduleName` |
| `createArtifact` | Creates an artifact repository entry | `artifactKey`, `groupId` |
| `deleteArtifact` | Deletes an artifact definition | `artifactKey` |
| `publishArtifactVersion` | Publishes files as a new artifact version | `artifactKey`, `artifactVersionState`, `version` |
| `retrieveArtifactVersions` | Downloads artifact versions to a workspace | `artifactKey`, `version` |
| `getArtifactVersion` | Returns metadata for a specific version | `artifactKey`, `version` |
| `getArtifactVersions` | Lists available versions for an artifact | `artifactKey` |
| `deleteArtifactVersion` | Deletes a specific artifact version | `artifactKey`, `version` |

---

## createSchedule

Creates a schedule that runs a procedure or pipeline on a cron-based or interval cadence. Schedules can also pass parameters to the target.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `scheduleName` | String | **Required.** Unique schedule name |
| `procedureName` | String | Procedure to run (mutually exclusive with `pipelineName`) |
| `pipelineName` | String | Pipeline to run |
| `description` | String | Schedule description |
| `cronExpression` | String | Standard 5-field cron expression (minute hour day month weekday) |
| `interval` | String | Run every N units (use with `intervalUnits`) |
| `intervalUnits` | String | `minutes`, `hours`, `days` |
| `startTime` | String | When the schedule becomes active (ISO-8601) |
| `stopTime` | String | When the schedule expires (ISO-8601) |
| `enabled` | Boolean | Whether the schedule is active (default true) |
| `timeZone` | String | Time zone for cron evaluation (e.g., `America/New_York`) |
| `actualParameter` | List | Parameters passed to each triggered run |
| `priority` | String | `low`, `normal`, `high` |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

// Nightly build at 2:00 AM Eastern, Monday through Friday
def result = ef.createSchedule(
    projectName:   "PaymentService",
    scheduleName:  "nightly-build",
    procedureName: "BuildAndTest",
    description:   "Nightly CI build — weekdays at 02:00 ET",
    cronExpression: "0 2 * * 1-5",
    timeZone:      "America/New_York",
    enabled:       true,
    priority:      "normal",
    actualParameter: [
        [actualParameterName: "branch",          value: "main"],
        [actualParameterName: "runTests",         value: "true"],
        [actualParameterName: "publishArtifact",  value: "true"]
    ]
)

if (!result) { throw new Exception("createSchedule failed") }
println "Created schedule: ${result.schedule.scheduleName}"
println "Schedule ID:      ${result.schedule.scheduleId}"
println "Next run:         ${result.schedule.nextRun}"
```

---

## modifySchedule / runSchedule

Update schedule attributes or trigger it immediately for on-demand execution.

### Key Named Parameters — modifySchedule

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `scheduleName` | String | **Required.** Schedule to modify |
| `enabled` | Boolean | Enable or disable the schedule |
| `cronExpression` | String | New cron expression |
| `actualParameter` | List | Updated parameter set |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

// Temporarily disable the nightly build during a maintenance window
ef.modifySchedule(
    projectName:  "PaymentService",
    scheduleName: "nightly-build",
    enabled:      false
)
println "Schedule disabled for maintenance window"

// Re-enable after maintenance
ef.modifySchedule(
    projectName:  "PaymentService",
    scheduleName: "nightly-build",
    enabled:      true
)
println "Schedule re-enabled"

// Trigger the schedule manually
def runResult = ef.runSchedule(
    projectName:  "PaymentService",
    scheduleName: "nightly-build"
)
println "Manual trigger — job: ${runResult.jobId}"

// List all schedules in the project
def schedules = ef.getSchedule(
    projectName:  "PaymentService",
    scheduleName: "nightly-build"
)
println "Next scheduled run: ${schedules.schedule.nextRun}"
```

---

## createArtifact

Registers an artifact in the CD/RO artifact repository.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `artifactKey` | String | **Required.** Artifact key in `groupId:artifactId` format |
| `groupId` | String | **Required.** Maven-style group identifier |
| `description` | String | Human-readable description |
| `repositoryName` | String | Repository to store versions in (default: `default`) |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def result = ef.createArtifact(
    artifactKey: "com.example.payment:payment-service",
    groupId:     "com.example.payment",
    description: "Payment service executable JAR artifact"
)

if (!result) { throw new Exception("createArtifact failed") }
println "Created artifact: ${result.artifact.artifactKey}"
println "Artifact ID:      ${result.artifact.artifactId}"
```

---

## publishArtifactVersion

Publishes one or more files as a new named version of an artifact.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `artifactKey` | String | **Required.** Target artifact in `groupId:artifactId` format |
| `version` | String | **Required.** Version string (e.g., `2.4.7-build.847`) |
| `artifactVersionState` | String | **Required.** `available` or `unavailable` |
| `repositoryName` | String | Repository to publish to (default: `default`) |
| `includePatterns` | String | Glob patterns of files to include |
| `excludePatterns` | String | Glob patterns to exclude |
| `fromDirectory` | String | Source directory containing files to publish |
| `description` | String | Version description |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def version  = ef.getProperty(propertyName: "/myJob/buildVersion").property.value
def buildDir = System.getenv("WORKSPACE") + "/target"

def result = ef.publishArtifactVersion(
    artifactKey:          "com.example.payment:payment-service",
    version:              version,
    artifactVersionState: "available",
    fromDirectory:        buildDir,
    includePatterns:      "payment-service-*.jar,payment-service-*.jar.sha256",
    description:          "Build ${version} from branch main, commit a3f8d91"
)

if (!result) { throw new Exception("publishArtifactVersion failed") }
println "Published artifact version: ${version}"
println "Artifact version ID: ${result.artifactVersion.artifactVersionId}"

ef.setProperty(
    propertyName: "/myJob/publishedArtifactVersion",
    value:        version
)
```

---

## retrieveArtifactVersions

Downloads artifact version files to the agent's workspace.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `artifactKey` | String | **Required.** Artifact to retrieve |
| `version` | String | **Required.** Version to download (supports ranges and `LATEST`) |
| `toDirectory` | String | Local directory to place downloaded files |
| `overwrite` | Boolean | Overwrite existing files (default true) |
| `repositoryName` | String | Source repository |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def version   = ef.getProperty(propertyName: "/myJob/publishedArtifactVersion").property.value
def deployDir = "/opt/payment-service/deploy"

def result = ef.retrieveArtifactVersions(
    artifactKey: "com.example.payment:payment-service",
    version:     version,
    toDirectory: deployDir,
    overwrite:   true
)

println "Downloaded artifact version ${version} to ${deployDir}"

def versions = ef.getArtifactVersions(
    artifactKey: "com.example.payment:payment-service"
)

println "Available versions:"
versions.artifactVersion.each { av ->
    println "  ${av.version}  state=${av.artifactVersionState}  created=${av.createTime}"
}
```

---

## deleteArtifactVersion

Removes a specific artifact version. Use in retention policies to reclaim storage.

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

// Retention policy: keep only the 10 most recent versions
def versions = ef.getArtifactVersions(
    artifactKey: "com.example.payment:payment-service"
)

def allVersions = versions.artifactVersion?.sort { it.createTime }
def toDelete    = allVersions?.dropRight(10) ?: []

toDelete.each { av ->
    ef.deleteArtifactVersion(
        artifactKey: "com.example.payment:payment-service",
        version:     av.version
    )
    println "Deleted old artifact version: ${av.version}"
}

println "Retention policy applied — kept ${Math.min(allVersions?.size() ?: 0, 10)} versions"
```