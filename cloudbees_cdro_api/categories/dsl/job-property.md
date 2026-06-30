# DSL Reference: Job, Property, PropertySheet

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/property
  https://docs.cloudbees.com/apidocs/dsl/latest/c/propertySheet
-->

## Running DSL

```bash
ectool evalDsl --dslFile script.groovy
ectool generateDsl --objectType project --objectName "WebApp"  # generate from existing
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `property` | Key/value metadata attached to any CD/RO object | `name`, `value` |
| `propertySheet` | Named container grouping multiple properties | `name` |

> Jobs are runtime objects (created when a procedure runs); they are not defined in DSL but are manipulated via API calls.

---

## property

### Closure Style (on a project)

```groovy
project "WebApp", {
    property "buildTool", value: "maven"
    property "artifactRepo", value: "https://nexus.example.com/repository/releases"
    property "releaseTrack", value: "stable"

    property "config", {
        description "Runtime configuration block"
        property "timeout",  value: "300"
        property "retries",  value: "3"
        property "logLevel", value: "INFO"
    }
}
```

### Closure Style (on a procedure)

```groovy
project "WebApp", {
    procedure "Build", {
        property "lastBuildTime", value: ""
        property "buildCount",    value: "0"
        property "metadata", {
            property "owner",  value: "platform-team"
            property "slack",  value: "#builds"
        }
    }
}
```

### API Call Style — setProperty / getProperty

```groovy
setProperty(
    propertyName: "/projects/WebApp/buildTool",
    value:        "maven"
)

setProperty(
    propertyName: "/myJob/results/testsPassed",
    value:        "142"
)

def prop = getProperty(propertyName: "/projects/WebApp/buildTool")
println prop.property.value

def val = getProperty(
    propertyName: "results/testsPassed",
    jobId:        "job-00001234"
).property.value

deleteProperty(propertyName: "/projects/WebApp/buildTool")

def sheet = getProperties(path: "/projects/WebApp")
sheet.propertySheet.property.each { p ->
    println "${p.propertyName}: ${p.value}"
}
```

### API Call Style — expandString

```groovy
def expanded = expandString(
    value:     'Building $[/projects/WebApp/artifactRepo]/artifact-$[version].jar',
    jobStepId: myJobStepId
).value
println expanded

def branch = expandString(value: '$[branch]').value
```

### YAML DSL (property on project)

```yaml
apiVersion: cloudbees.com/v2023.2
kind: project
metadata:
  name: WebApp
spec:
  properties:
    - propertyName: buildTool
      value: maven
    - propertyName: artifactRepo
      value: https://nexus.example.com/repository/releases
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `value` | String | Yes (for leaf) | String value of the property |
| `description` | String | No | Documentation string |
| `expandable` | Boolean | No | Whether `$[...]` references expand in value |

---

## Job API Calls (Runtime Operations)

Jobs are not DSL-defined objects; they are created when `runProcedure` or a schedule fires.

### getJob / getJobs

```groovy
def job = getJob(jobId: "job-00001234")
println job.job.status
println job.job.outcome
println job.job.start
println job.job.finish
println job.job.jobName

def jobs = getJobs(
    projectName:   "WebApp",
    procedureName: "Build",
    maxIds:        10,
    jobStatus:     "completed"
)
jobs.job.each { j ->
    println "${j.jobId}: ${j.outcome}"
}
```

### abortJob

```groovy
abortJob(
    jobId: "job-00001234",
    force: true
)
```

### waitForJob

```groovy
def job = waitForJob(
    jobId:   "job-00001234",
    timeout: 3600
)
if (job.job.outcome == "error") {
    error "Job failed: ${job.job.jobId}"
}
```

### getJobStepLog

```groovy
def log = getJobStepLog(
    jobStepId: "jobstep-000045678"
)
println log.log

def steps = getJobSteps(jobId: "job-00001234")
def compileStep = steps.jobStep.find { it.stepName == "Compile" }
println getJobStepLog(jobStepId: compileStep.jobStepId).log
```

### Full Runtime Workflow Example

```groovy
def run = runProcedure(
    projectName:   "WebApp",
    procedureName: "Build",
    actualParameter: [
        [actualParameterName: "branch",    value: "release/3.0"],
        [actualParameterName: "skipTests", value: "false"]
    ]
)
def jobId = run.jobId
println "Launched job: ${jobId}"

def job = waitForJob(jobId: jobId, timeout: 1800)
println "Outcome: ${job.job.outcome}"

def version = getProperty(
    propertyName: "artifactVersion",
    jobId:        jobId
).property.value
println "Built artifact version: ${version}"

if (job.job.outcome == "error") {
    def steps = getJobSteps(jobId: jobId)
    steps.jobStep.findAll { it.outcome == "error" }.each { s ->
        println "=== Failed step: ${s.stepName} ==="
        println getJobStepLog(jobStepId: s.jobStepId).log
    }
}

setProperty(
    propertyName: "/jobs/${jobId}/review/status",
    value:        "passed"
)
```