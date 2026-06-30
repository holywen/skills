# CloudBees CD/RO Groovy API — Project, Procedure & Step

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
// ec-groovy -DCOMMANDER_SERVER=cdro.example.com -DCOMMANDER_SECURE=1 script.groovy
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createProject` | Creates a new project | `projectName` |
| `deleteProject` | Deletes a project | `projectName` |
| `modifyProject` | Updates project attributes | `projectName` |
| `getProject` | Returns project details | `projectName` |
| `getProjects` | Lists all projects | — |
| `createProcedure` | Creates a procedure | `projectName`, `procedureName` |
| `deleteProcedure` | Deletes a procedure | `projectName`, `procedureName` |
| `modifyProcedure` | Updates procedure attributes | `projectName`, `procedureName` |
| `runProcedure` | Runs a procedure | `projectName`, `procedureName` |
| `copyProcedure` | Copies a procedure | `projectName`, `procedureName`, `cloneProjectName`, `cloneProcedureName` |
| `createStep` | Creates a step in a procedure | `projectName`, `procedureName`, `stepName` |
| `deleteStep` | Deletes a step | `projectName`, `procedureName`, `stepName` |
| `modifyStep` | Updates step attributes | `projectName`, `procedureName`, `stepName` |
| `getSteps` | Lists steps in a procedure | `projectName`, `procedureName` |

---

### `createProject`

```groovy
def result = ef.createProject(
    projectName:          "PaymentService",
    description:          "Payment microservice CI/CD",
    defaultResourceName:  "linux-build-pool",
    defaultWorkspaceName: "default"
)
println "Created: ${result.project.projectName}"
```

---

### `runProcedure`

```groovy
def result = ef.runProcedure(
    projectName:   "PaymentService",
    procedureName: "BuildAndTest",
    actualParameter: [
        [actualParameterName: "branch",      value: "feature/payment-v2"],
        [actualParameterName: "environment", value: "staging"],
    ],
    priority: "high"
)
def jobId = result.jobId
println "Job started: ${jobId}"

def finalStatus = ef.waitForJob(jobId: jobId, pollInterval: 10, timeout: 3600)
if (finalStatus.outcome == "error") throw new Exception("Build failed")
```

---

### `createStep`

```groovy
// Command step
ef.createStep(
    projectName:    "PaymentService",
    procedureName:  "BuildAndTest",
    stepName:       "Compile",
    shell:          "bash",
    command:        "mvn clean compile -B -q",
    resourceName:   "linux-build-pool",
    timeLimit:      "15",
    timeLimitUnits: "minutes",
    errorHandling:  "failProcedure"
)

// Subprocedure step
ef.createStep(
    projectName:   "PaymentService",
    procedureName: "BuildAndTest",
    stepName:      "PublishArtifact",
    subproject:    "SharedLibrary",
    subprocedure:  "PublishMavenArtifact",
    actualParameter: [
        [actualParameterName: "version", value: "\$[/myJob/buildVersion]"]
    ]
)
```