# CloudBees CD/RO Groovy API — Application & Component

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createApplication` | Creates an application model | `projectName`, `applicationName` |
| `deleteApplication` | Deletes an application | `projectName`, `applicationName` |
| `modifyApplication` | Updates application attributes | `projectName`, `applicationName` |
| `deployApplication` | Deploys to an environment | `projectName`, `applicationName`, `environmentName` |
| `createComponent` | Creates a deployable component | `projectName`, `applicationName`, `componentName` |
| `modifyComponent` | Updates component attributes | `projectName`, `applicationName`, `componentName` |
| `createProcess` | Creates a deploy/undeploy process | `projectName`, `applicationName`, `processName` |
| `modifyProcess` | Updates process attributes | `projectName`, `applicationName`, `processName` |
| `createProcessStep` | Adds a step to a process | `projectName`, `applicationName`, `processName`, `processStepName` |
| `runProcess` | Runs an application process | `projectName`, `applicationName`, `processName`, `environmentName` |

---

### `createApplication` + `createComponent`

```groovy
ef.createApplication(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    description:     "RESTful payment processing API"
)

ef.createComponent(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    componentName:   "payment-service-jar",
    description:     "Core payment service executable JAR"
)

ef.createComponent(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    componentName:   "db-migrations",
    description:     "Flyway database migration scripts"
)
```

---

### `createProcess` + `createProcessStep`

```groovy
ef.createProcess(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    processName:     "Deploy",
    processType:     "DEPLOY",
    description:     "Full application deploy: migrations then service"
)

// Step 1: DB migrations
ef.createProcessStep(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    processName:     "Deploy",
    processStepName: "RunMigrations",
    subcomponent:    "db-migrations",
    subcomponentApplicationName: "payment-api",
    subcomponentProcess: "Deploy",
    errorHandling:   "stopOnError"
)

// Step 2: deploy service JAR
ef.createProcessStep(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    processName:     "Deploy",
    processStepName: "DeployService",
    subcomponent:    "payment-service-jar",
    subcomponentApplicationName: "payment-api",
    subcomponentProcess: "Deploy",
    errorHandling:   "stopOnError"
)
```

---

### `deployApplication`

```groovy
def result = ef.deployApplication(
    projectName:     "PaymentService",
    applicationName: "payment-api",
    environmentName: "staging",
    processName:     "Deploy",
    actualParameter: [
        [actualParameterName: "artifactVersion", value: "2.4.7-build.847"],
        [actualParameterName: "configProfile",   value: "staging"]
    ]
)
def jobId = result.jobId
println "Deployment job: ${jobId}"

def finalStatus = ef.waitForJob(jobId: jobId)
if (finalStatus.outcome == "error") {
    throw new Exception("Deployment failed: ${jobId}")
}
println "Deployment completed successfully"
```