# CloudBees CD/RO Groovy API — Workspace, Workflow, Catalog & Server

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`
> Example: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/createWorkspace`

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
| `createWorkspace` | Creates a named workspace (file staging area) | `workspaceName` |
| `modifyWorkspace` | Updates workspace attributes | `workspaceName` |
| `createGateway` | Creates a gateway for zone-to-zone communication | `gatewayName` |
| `modifyGateway` | Updates gateway attributes | `gatewayName` |
| `createZone` | Creates a network zone | `zoneName` |
| `modifyZone` | Updates zone attributes | `zoneName` |
| `runWorkflow` | Starts a workflow instance | `projectName`, `workflowName` |
| `completeWorkflow` | Completes a running workflow | `projectName`, `workflowName`, `workflowId` |
| `transitionWorkflow` | Moves a workflow to a named state | `projectName`, `workflowName`, `workflowId`, `transitionName` |
| `createWorkflowDefinition` | Creates a workflow definition | `projectName`, `workflowDefinitionName` |
| `createStateDefinition` | Creates a state in a workflow definition | `projectName`, `workflowDefinitionName`, `stateDefinitionName` |
| `createTransitionDefinition` | Creates a transition between workflow states | `projectName`, `workflowDefinitionName`, `transitionDefinitionName` |
| `createScmSync` | Creates a source-control sync configuration | `projectName`, `procedureName` |
| `runScmSync` | Triggers an SCM sync | `projectName`, `procedureName` |
| `createTrigger` | Creates a webhook or polling trigger | `projectName`, `procedureName`, `triggerName` |
| `runTrigger` | Fires a trigger manually | `projectName`, `procedureName`, `triggerName` |
| `createCatalog` | Creates a service catalog | `projectName`, `catalogName` |
| `createCatalogItem` | Adds an item to a service catalog | `projectName`, `catalogName`, `catalogItemName` |
| `runCatalogItem` | Runs a catalog item | `projectName`, `catalogName`, `catalogItemName` |
| `evalDsl` | Evaluates a DSL string on the server | `dslString` |
| `findObjects` | Searches for CD/RO objects by type and filter | `objectType` |
| `export` | Exports project or objects as DSL | `projectName` |
| `generateDsl` | Generates DSL from existing objects | `objectType`, `objectName` |
| `clone` | Clones a project or object | `projectName`, `cloneProjectName` |
| `getServerStatus` | Returns server health and version info | — |
| `getVersions` | Returns component version details | — |
| `sendReportingData` | Sends DevOps Insight reporting events | `payload`, `reportObjectTypeName` |

---

## createWorkspace / modifyWorkspace

Workspaces define where job step artifacts are staged on agent hosts.

### Key Named Parameters — createWorkspace

| Parameter | Type | Description |
|---|---|---|
| `workspaceName` | String | **Required.** Unique workspace name |
| `description` | String | Human-readable description |
| `agentDrivePath` | String | Path on Windows agent hosts |
| `agentUncPath` | String | UNC path for Windows agents |
| `agentUnixPath` | String | Path on Unix/Linux agent hosts |
| `local` | Boolean | Restrict workspace to the local agent only |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def result = ef.createWorkspace(
    workspaceName:  "payment-service-ws",
    description:    "Workspace for payment service build artifacts",
    agentUnixPath:  "/opt/cloudbees/workspaces/payment-service",
    agentDrivePath: "C:/cloudbees/workspaces/payment-service",
    local:          false
)

println "Created workspace: ${result.workspace.workspaceName}"
println "Workspace ID:      ${result.workspace.workspaceId}"
println "Unix path:         ${result.workspace.agentUnixPath}"

ef.modifyWorkspace(
    workspaceName: "payment-service-ws",
    agentUnixPath: "/mnt/shared/workspaces/payment-service"
)
```

---

## createZone / createGateway

Zones group resources by network segment. Gateways bridge communication between zones.

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createZone(
    zoneName:    "us-east-production",
    description: "Production VPC — US East region"
)

ef.createZone(
    zoneName:    "us-east-staging",
    description: "Staging VPC — US East region"
)

ef.createGateway(
    gatewayName:         "prod-zone-gateway",
    description:         "Gateway from CD server to production VPC",
    gatewayResourceName: "gateway-server-side",
    agentResourceName:   "gateway-prod-vpc-side"
)

println "Zone and gateway topology configured"
```

---

## evalDsl

Evaluates a Groovy DSL string directly on the CD/RO server. The most powerful imperative tool for bulk provisioning and templating.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `dslString` | String | **Required.** DSL code to evaluate |
| `parameters` | String | JSON string of parameters accessible as `args` inside DSL |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow
import groovy.json.JsonOutput

ElectricFlow ef = new ElectricFlow()

def dsl = """
project 'MicroservicePlatform', {
    description = 'Auto-provisioned microservice platform'

    ['auth-service', 'catalog-service', 'notification-service'].each { svcName ->
        procedure svcName + '-Build', {
            description = "CI build for " + svcName
            step 'Compile', {
                shell   = 'bash'
                command = 'mvn clean package -B -q'
            }
            step 'Test', {
                shell   = 'bash'
                command = 'mvn test -B'
            }
        }
    }
}
"""

def result = ef.evalDsl(dslString: dsl)
println "DSL evaluated successfully"
println result

def params = JsonOutput.toJson([environment: "staging", region: "us-east-1"])
def paramDsl = """
def env    = args.environment
def region = args.region
setProperty propertyName: '/projects/MicroservicePlatform/deployRegion',
            value:        region
"""

ef.evalDsl(
    dslString:  paramDsl,
    parameters: params
)
```

---

## findObjects

Searches for CD/RO objects by type with optional filter criteria.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `objectType` | String | **Required.** `project`, `procedure`, `job`, `pipeline`, `resource`, `environment`, `application`, etc. |
| `filter` | List | Filter criteria maps |
| `maxIds` | Integer | Limit results |
| `firstResult` | Integer | Pagination offset |
| `sortKey` | String | Property to sort by |
| `sortOrder` | String | `ascending` or `descending` |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def failedJobs = ef.findObjects(
    objectType: "job",
    filter: [
        [propertyName: "outcome",    operator: "equals",      operand1: "error"],
        [propertyName: "createTime", operator: "greaterThan", operand1: new Date(System.currentTimeMillis() - 86400000).format("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")]
    ],
    sortKey:   "createTime",
    sortOrder: "descending",
    maxIds:    50
)

println "Failed jobs in last 24h: ${failedJobs.object?.size() ?: 0}"
failedJobs.object?.each { obj ->
    println "  ${obj.job.jobName}  project=${obj.job.projectName}  started=${obj.job.start}"
}

def offlineResources = ef.findObjects(
    objectType: "resource",
    filter: [
        [propertyName: "agentState", operator: "notEqual", operand1: "alive"]
    ]
)

offlineResources.object?.each { obj ->
    println "OFFLINE: ${obj.resource.resourceName}  host=${obj.resource.hostName}"
}
```

---

## export / generateDsl

Export project definitions as DSL for version control, backup, or migration.

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def exportResult = ef.export(
    projectName: "PaymentService",
    includeAcls: true
)

new File("/workspace/cdro-dsl/PaymentService.groovy").text = exportResult.dsl
println "Project DSL exported (${exportResult.dsl?.length()} chars)"

def pipelineDsl = ef.generateDsl(
    objectType:   "pipeline",
    objectName:   "Release-Pipeline",
    projectName:  "PaymentService",
    withChildren: true
)

println "Pipeline DSL:"
println pipelineDsl.dsl
```

---

## getServerStatus / getVersions

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def status = ef.getServerStatus()
println "Server status:  ${status.serverStatus.status}"
println "Server version: ${status.serverStatus.version}"
println "Uptime:         ${status.serverStatus.uptime}"
println "License valid:  ${status.serverStatus.licenseValid}"

def versions = ef.getVersions()
versions.version.each { v ->
    println "${v.component}: ${v.version}"
}
```

---

## createCatalog / createCatalogItem / runCatalogItem

The service catalog exposes self-service actions to end users.

### Key Named Parameters — createCatalogItem

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `catalogName` | String | **Required.** Parent catalog |
| `catalogItemName` | String | **Required.** Item name |
| `description` | String | What this catalog item does |
| `procedureName` | String | Procedure to run when item is invoked |
| `pipelineName` | String | Pipeline to run |
| `buttonLabel` | String | Label on the run button |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createCatalog(
    projectName:  "PaymentService",
    catalogName:  "Developer Self-Service",
    description:  "Self-service actions for the payment service team"
)

ef.createCatalogItem(
    projectName:     "PaymentService",
    catalogName:     "Developer Self-Service",
    catalogItemName: "Deploy to Staging",
    description:     "Deploy a specific artifact version to the staging environment",
    procedureName:   "DeployToEnvironment",
    buttonLabel:     "Deploy"
)

ef.createCatalogItem(
    projectName:     "PaymentService",
    catalogName:     "Developer Self-Service",
    catalogItemName: "Run Smoke Tests",
    description:     "Execute smoke tests against staging",
    procedureName:   "SmokeTests",
    buttonLabel:     "Run Tests"
)

def runResult = ef.runCatalogItem(
    projectName:     "PaymentService",
    catalogName:     "Developer Self-Service",
    catalogItemName: "Deploy to Staging",
    actualParameter: [
        [actualParameterName: "artifactVersion", value: "2.4.7-build.847"],
        [actualParameterName: "environment",     value: "staging"]
    ]
)

println "Catalog item run — jobId: ${runResult.jobId}"
```

---

## createTrigger / runTrigger

Triggers fire procedures or pipelines in response to external events.

### Key Named Parameters — createTrigger

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `procedureName` | String | Procedure to trigger |
| `pipelineName` | String | Pipeline to trigger |
| `triggerName` | String | **Required.** Unique trigger name |
| `triggerType` | String | `webhook`, `pollingScm`, `schedule` |
| `enabled` | Boolean | Whether the trigger is active |
| `quietTime` | Integer | Seconds to wait after event before firing |
| `actualParameter` | List | Parameters passed when trigger fires |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createTrigger(
    projectName:   "PaymentService",
    pipelineName:  "Release-Pipeline",
    triggerName:   "github-push",
    triggerType:   "webhook",
    enabled:       true,
    quietTime:     30,
    actualParameter: [
        [actualParameterName: "branch", value: "\$[/myTrigger/branch]"]
    ]
)

println "Webhook trigger created"

def fireResult = ef.runTrigger(
    projectName:  "PaymentService",
    pipelineName: "Release-Pipeline",
    triggerName:  "github-push"
)
println "Trigger fired — flowRuntimeId: ${fireResult.flowRuntime?.flowRuntimeId}"
```

---

## clone

Clones a project to a new name. Useful for creating per-release copies.

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

def result = ef.clone(
    projectName:      "PaymentService",
    cloneProjectName: "PaymentService-v2.5-branch",
    description:      "Working copy for v2.5 feature development"
)

println "Cloned project: ${result.project.projectName}"
```