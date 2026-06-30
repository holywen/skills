# CloudBees CD/RO Groovy API — Pipeline, Release & Stage

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createPipeline` | Creates a new pipeline | `projectName`, `pipelineName` |
| `deletePipeline` | Deletes a pipeline | `projectName`, `pipelineName` |
| `modifyPipeline` | Updates pipeline attributes | `projectName`, `pipelineName` |
| `runPipeline` | Starts a pipeline run | `projectName`, `pipelineName` |
| `abortPipelineRun` | Aborts a running pipeline | `flowRuntimeId` |
| `pausePipelineRun` | Pauses a running pipeline | `flowRuntimeId` |
| `resumePipelineRun` | Resumes a paused pipeline | `flowRuntimeId` |
| `getPipelineRuntimes` | Lists historical pipeline runs | `projectName`, `pipelineName` |
| `createRelease` | Creates a release | `projectName`, `releaseName` |
| `startRelease` | Starts a release | `projectName`, `releaseName` |
| `completeRelease` | Marks a release complete | `projectName`, `releaseName` |
| `createStage` | Creates a pipeline stage | `projectName`, `pipelineName`, `stageName` |
| `createTask` | Creates a task in a stage | `projectName`, `pipelineName`, `stageName`, `taskName` |
| `completeManualTask` | Completes a manual approval task | `flowRuntimeId`, `stageName`, `taskName` |
| `createGate` | Creates an entry or exit gate | `projectName`, `pipelineName`, `stageName`, `gateType` |

---

### `createPipeline` + stages + tasks

```groovy
ef.createPipeline(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    description:  "Build → Test → Staging → Production"
)

["Build", "Integration-Test", "Staging-Deploy", "Production-Deploy"].each { stage ->
    ef.createStage(
        projectName:  "PaymentService",
        pipelineName: "Release-Pipeline",
        stageName:    stage
    )
}

// Procedure task
ef.createTask(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    stageName:    "Build",
    taskName:     "CompileAndPackage",
    taskType:     "PROCEDURE",
    subproject:   "PaymentService",
    subprocedure: "BuildAndTest",
    actualParameter: [
        [actualParameterName: "branch", value: "\$[branch]"]
    ],
    errorHandling: "stopOnError"
)

// Manual approval task
ef.createTask(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    stageName:    "Production-Deploy",
    taskName:     "ProductionApproval",
    taskType:     "MANUAL",
    instruction:  "Review staging results and approve production release.",
    approverName: ["release-manager", "ops-lead"]
)
```

---

### `runPipeline`

```groovy
def result = ef.runPipeline(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    actualParameter: [
        [actualParameterName: "branch",      value: "release/2.4.7"],
        [actualParameterName: "buildNumber", value: "847"]
    ]
)
def flowRuntimeId = result.flowRuntime.flowRuntimeId
println "Pipeline started: ${flowRuntimeId}"
```

---

### `completeManualTask`

```groovy
ef.completeManualTask(
    flowRuntimeId: "flowRuntime-00012345-6789-abcd-ef01-234567890abc",
    stageName:     "Production-Deploy",
    taskName:      "ProductionApproval",
    decision:      "approve",
    comment:       "Staging tests passed — approved for production."
)
```

---

### `createRelease` + `startRelease`

```groovy
ef.createRelease(
    projectName:      "PaymentService",
    releaseName:      "v2.4.7",
    pipelineName:     "Release-Pipeline",
    description:      "Payment service v2.4.7",
    plannedStartDate: "2024-07-01",
    plannedEndDate:   "2024-07-15"
)

def startResult = ef.startRelease(
    projectName: "PaymentService",
    releaseName: "v2.4.7"
)
println "Release started: ${startResult.flowRuntime?.flowRuntimeId}"
```

---

### `createGate`

```groovy
ef.createGate(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    stageName:    "Staging-Deploy",
    gateType:     "POST"
)

ef.createTask(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    stageName:    "Staging-Deploy",
    taskName:     "SmokeTestGate",
    taskType:     "PROCEDURE",
    gateType:     "POST",
    subproject:   "PaymentService",
    subprocedure: "SmokeTests"
)
```