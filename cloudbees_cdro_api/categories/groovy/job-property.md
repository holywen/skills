# CloudBees CD/RO Groovy API — Job & Property

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `getJob` | Returns details of a single job | `jobId` |
| `getJobs` | Lists jobs, optionally filtered | — |
| `abortJob` | Aborts a running job | `jobId` |
| `deleteJob` | Deletes a completed job record | `jobId` |
| `waitForJob` | Blocks until job completes | `jobId` |
| `getJobStatus` | Returns current job status | `jobId` |
| `getJobStep` | Returns details of a job step | `jobStepId` |
| `getJobStepLog` | Returns log output of a job step | `jobStepId` |
| `setProperty` | Sets a property value | `propertyName`, `value` |
| `getProperty` | Gets a property value | `propertyName` |
| `getProperties` | Lists properties under a path | `path` |
| `deleteProperty` | Deletes a property | `propertyName` |
| `expandString` | Expands property references in a string | `value` |

---

### `waitForJob`

```groovy
def runResult = ef.runProcedure(
    projectName:   "PaymentService",
    procedureName: "BuildAndTest",
    actualParameter: [
        [actualParameterName: "branch", value: "main"]
    ]
)
def jobId = runResult.jobId

def finalStatus = ef.waitForJob(
    jobId:        jobId,
    pollInterval: 10,
    timeout:      3600
)
if (finalStatus.outcome == "error") {
    throw new Exception("Job failed: ${jobId}")
}
println "Job succeeded: ${finalStatus.jobName}"
```

---

### `getJobStepLog`

```groovy
def jobResult = ef.getJob(jobId: "job-00012345-6789-abcd-ef01-234567890abc")
jobResult.job.jobStep.each { step ->
    println "=== ${step.stepName} (${step.outcome}) ==="
    def logResult = ef.getJobStepLog(jobStepId: step.jobStepId)
    println logResult.log ?: "(no output)"
}
```

---

### `setProperty` / `getProperty`

```groovy
// Write properties during a build step
ef.setProperty(
    propertyName: "/myJob/buildVersion",
    value:        "2.4.7-build.1042"
)
ef.setProperty(
    propertyName: "/myJob/gitCommit",
    value:        "a3f8d912c4b7e5f2"
)

// Read back
def version = ef.getProperty(propertyName: "/myJob/buildVersion")
println "Build version: ${version.property.value}"

// Project-level property
def configProp = ef.getProperty(propertyName: "/projects/PaymentService/deployTarget")
println "Deploy target: ${configProp.property.value}"
```

---

### `expandString`

```groovy
def expanded = ef.expandString(
    value: "Deploy \$[/myJob/buildVersion] to \$[/myJob/deployTarget]",
    jobId: "job-00012345-6789-abcd-ef01-234567890abc"
)
println "Expanded: ${expanded.value}"
```