# DSL Reference: Workspace, Gateway, Zone, Workflow, Catalog, Server Objects

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/workspace
  https://docs.cloudbees.com/apidocs/dsl/latest/c/gateway
  https://docs.cloudbees.com/apidocs/dsl/latest/c/zone
  https://docs.cloudbees.com/apidocs/dsl/latest/c/workflowDefinition
  https://docs.cloudbees.com/apidocs/dsl/latest/c/stateDefinition
  https://docs.cloudbees.com/apidocs/dsl/latest/c/transitionDefinition
  https://docs.cloudbees.com/apidocs/dsl/latest/c/trigger
  https://docs.cloudbees.com/apidocs/dsl/latest/c/scmSync
  https://docs.cloudbees.com/apidocs/dsl/latest/c/catalog
  https://docs.cloudbees.com/apidocs/dsl/latest/c/catalogItem
-->

## Running DSL

```bash
ectool evalDsl --dslFile objects.groovy
ectool generateDsl --objectType workspace           --objectName "default"
ectool generateDsl --objectType catalog             --objectName "DevOps Catalog" --projectName "WebApp"
ectool generateDsl --objectType workflowDefinition  --objectName "ApprovalFlow"   --projectName "WebApp"
ectool export --projectName "WebApp" --includeAllChildren true > webapp.dsl
ectool findObjects --objectType procedure --queryString "projectName=WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `workspace` | Named working directory on an agent for step execution | `name` |
| `gateway` | Proxy agent enabling communication to agents in isolated networks | `name` |
| `zone` | Logical grouping of resources behind a gateway | `name` |
| `workflowDefinition` | State-machine definition for approval or custom workflows | `name` |
| `stateDefinition` | Single state within a workflow definition | `name` |
| `transitionDefinition` | Named transition between workflow states | `name` |
| `trigger` | Event-based (SCM, webhook) trigger for procedures or pipelines | `name` |
| `scmSync` | SCM synchronization definition for a project | (project-level) |
| `catalog` | Self-service catalog of reusable pipeline/procedure templates | `name` |
| `catalogItem` | Single item (template entry) within a catalog | `name` |

---

## workspace

### Closure Style

```groovy
workspace "build-ws", {
    description    "Shared build workspace on linux agents"
    local          false
    agentDrivePath "/workspace/build"
    cleanupPolicy  "afterJob"
    windows        false
}
```

### API Call Style

```groovy
createWorkspace(
    workspaceName:  "build-ws",
    description:    "Shared build workspace",
    local:          false,
    agentDrivePath: "/workspace/build",
    cleanupPolicy:  "afterJob"
)

modifyWorkspace(
    workspaceName: "build-ws",
    cleanupPolicy: "afterStep"
)

deleteWorkspace(workspaceName: "build-ws")
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `agentDrivePath` | String | No | Absolute path on the agent |
| `local` | Boolean | No | `true` = private per job; `false` = shared |
| `cleanupPolicy` | String | No | `none`, `afterStep`, `afterJob` |
| `windows` | Boolean | No | Use Windows path conventions |

---

## gateway

```groovy
gateway "dmz-gateway", {
    description  "Gateway for DMZ network segment"
    hostName     "gateway-dmz.internal.example.com"
    port         7800
    resourceName "gateway-agent-dmz"
}
```

---

## zone

```groovy
zone "dmz-zone", {
    description "All resources in the DMZ network"
    gatewayName "dmz-gateway"

    resource "dmz-agent-01", {
        hostName "dmz-agent-01.dmz.internal"
        port     7800
    }

    resource "dmz-agent-02", {
        hostName "dmz-agent-02.dmz.internal"
        port     7800
    }
}
```

---

## workflowDefinition / stateDefinition / transitionDefinition

### Closure Style

```groovy
project "WebApp", {
    workflowDefinition "ReleaseApproval", {
        description "Multi-stage release approval workflow"

        stateDefinition "Submitted", {
            type        "start"
            description "Ticket submitted; awaiting triage"

            transitionDefinition "to-review", {
                stateDefinitionName "Under Review"
                condition           '$[/javascript]true'
            }
        }

        stateDefinition "Under Review", {
            type             "normal"
            onEnterProcedure "WebApp:NotifyReviewers"
            onExitProcedure  "WebApp:LogTransition"

            transitionDefinition "approve", { stateDefinitionName "Approved" }
            transitionDefinition "reject",  { stateDefinitionName "Rejected" }
        }

        stateDefinition "Approved", {
            type             "normal"
            onEnterProcedure "WebApp:TriggerDeploy"
            transitionDefinition "complete", { stateDefinitionName "Closed" }
        }

        stateDefinition "Rejected", {
            type             "normal"
            onEnterProcedure "WebApp:NotifyRejection"
            transitionDefinition "resubmit", { stateDefinitionName "Submitted" }
            transitionDefinition "close",    { stateDefinitionName "Closed" }
        }

        stateDefinition "Closed", { type "end" }
    }
}
```

### API Call Style

```groovy
def wf = runWorkflow(
    projectName:            "WebApp",
    workflowDefinitionName: "ReleaseApproval",
    actualParameter: [
        [actualParameterName: "ticketId", value: "JIRA-1234"]
    ]
)
println "Workflow run: ${wf.workflow.workflowId}"

transitionWorkflow(
    workflowId:               wf.workflow.workflowId,
    transitionDefinitionName: "to-review"
)

def state = getWorkflow(workflowId: wf.workflow.workflowId)
println "Current state: ${state.workflow.currentState}"
```

### Key Properties — stateDefinition

| Property | Type | Required | Description |
|---|---|---|---|
| `type` | String | Yes | `start`, `end`, `normal` |
| `description` | String | No | State description |
| `onEnterProcedure` | String | No | `project:procedure` called on state entry |
| `onExitProcedure` | String | No | `project:procedure` called on state exit |

---

## trigger

### Closure Style — SCM polling

```groovy
project "WebApp", {
    procedure "Build", {
        trigger "On-Push", {
            description   "Trigger on Git push to main"
            triggerType   "polling"
            quietPeriod   60
            interval      "5"
            intervalUnits "minutes"
            enabled       true

            actualParameter [
                [actualParameterName: "branch", value: "main"]
            ]
        }
    }
}
```

### Closure Style — webhook trigger on a pipeline

```groovy
project "WebApp", {
    pipeline "CI-CD", {
        trigger "Webhook-Trigger", {
            description "Incoming webhook from GitHub Actions"
            triggerType "webhook"

            actualParameter [
                [actualParameterName: "branch", value: '$[/myTrigger/webhookData/ref]']
            ]
        }
    }
}
```

---

## scmSync

```groovy
project "WebApp", {
    scmSync {
        scmType        "git"
        repositoryUrl  "https://github.com/example/webapp-cdro.git"
        branch         "main"
        localDirectory "/workspace/scm-sync/webapp"
        credential     "github-token"
        syncSchedule   "*/5 * * * *"
        enabled        true
    }
}
```

---

## catalog / catalogItem

### Closure Style

```groovy
project "WebApp", {
    catalog "DevOps Catalog", {
        description "Self-service catalog for WebApp team"

        catalogItem "Deploy Application", {
            description         "Deploy a versioned artifact to any environment"
            buttonLabel         "Deploy"
            endTargetJson       '{"endpoint":"WebApp","type":"procedure","name":"Deploy"}'
            useFormalParameters true

            formalParameter "version", {
                required     true
                type         "entry"
                description  "Artifact version to deploy"
            }

            formalParameter "environment", {
                required     true
                type         "select"
                options {
                    option "Staging",    { value "stg" }
                    option "Production", { value "prd" }
                }
            }
        }

        catalogItem "Run Regression Suite", {
            description         "Execute full regression test suite"
            buttonLabel         "Run Tests"
            endTargetJson       '{"endpoint":"WebApp","type":"procedure","name":"FullRegression"}'
            useFormalParameters true

            formalParameter "testSuite", {
                type         "select"
                required     true
                options {
                    option "Smoke",      { value "smoke" }
                    option "Regression", { value "regression" }
                    option "Full",       { value "full" }
                }
                defaultValue "smoke"
            }
        }
    }
}
```

### API Call Style

```groovy
createCatalog(
    projectName:  "WebApp",
    catalogName:  "DevOps Catalog",
    description:  "Self-service catalog for WebApp team"
)

createCatalogItem(
    projectName:         "WebApp",
    catalogName:         "DevOps Catalog",
    catalogItemName:     "Deploy Application",
    description:         "Deploy artifact to any environment",
    buttonLabel:         "Deploy",
    endTargetJson:       '{"endpoint":"WebApp","type":"procedure","name":"Deploy"}',
    useFormalParameters: true
)

def run = runCatalogItem(
    projectName:     "WebApp",
    catalogName:     "DevOps Catalog",
    catalogItemName: "Deploy Application",
    actualParameter: [
        [actualParameterName: "version",     value: "3.0.1"],
        [actualParameterName: "environment", value: "stg"]
    ]
)
println "Launched job: ${run.jobId}"
```

---

## Server-Level API Calls: evalDsl, findObjects, export, generateDsl

### evalDsl

```groovy
evalDsl(dsl: 'project "TestProject", { description "from evalDsl" }')

evalDsl(dslFile: "/workspace/dsl/setup.groovy")

evalDsl(
    dslFile: "/workspace/dsl/deploy.groovy",
    parameters: [
        [parameterName: "version",     parameterValue: "3.0.1"],
        [parameterName: "environment", parameterValue: "stg"]
    ]
)
```

### findObjects

```groovy
def procs = findObjects(
    objectType:  "procedure",
    queryString: "projectName=WebApp"
)
procs.object.each { o -> println o.objectName }

def jobs = findObjects(
    objectType:  "job",
    queryString: "createTime > '2025-12-01T00:00:00Z' and projectName=WebApp"
)
```

### export / generateDsl

```groovy
def dsl = generateDsl(
    objectType:         "project",
    objectName:         "WebApp",
    includeAllChildren: true,
    includeCredentials: false
)
new File("/backup/webapp.dsl").text = dsl.dsl

def procDsl = generateDsl(
    objectType:       "procedure",
    objectName:       "Build",
    projectName:      "WebApp",
    suppressDefaults: true
)
println procDsl.dsl
```