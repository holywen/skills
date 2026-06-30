# DSL Reference: Pipeline, Release, Stage, Task, Gate

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/pipeline
  https://docs.cloudbees.com/apidocs/dsl/latest/c/release
  https://docs.cloudbees.com/apidocs/dsl/latest/c/stage
  https://docs.cloudbees.com/apidocs/dsl/latest/c/task
  https://docs.cloudbees.com/apidocs/dsl/latest/c/gate
-->

## Running DSL

```bash
ectool evalDsl --dslFile pipeline.groovy
ectool generateDsl --objectType pipeline --objectName "CI-CD" --projectName "WebApp"
ectool generateDsl --objectType release  --objectName "Release-3.0" --projectName "WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `pipeline` | Sequential or parallel stages forming a CI/CD workflow | `name` |
| `stage` | Phase within a pipeline (Build, Test, Deploy, etc.) | `name` |
| `task` | Work item within a stage | `name`, `taskType` |
| `gate` | Entry/exit condition set evaluated before/after a stage | `condition` |
| `release` | Managed delivery process coordinating pipelines and approvals | `name` |

---

## pipeline

### Closure Style

```groovy
project "WebApp", {
    pipeline "CI-CD", {
        description "Full CI/CD pipeline from build to production"
        enabled     true

        formalParameter "branch", {
            defaultValue "main"
            required     true
            type         "entry"
        }

        formalParameter "targetEnv", {
            type         "select"
            defaultValue "staging"
            options {
                option "Staging",    { value "staging" }
                option "Production", { value "production" }
            }
        }

        stage "Build", {
            colorCode "#0078d4"
            task "Compile and Package", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "Build"
                actualParameter [
                    [actualParameterName: "branch",    value: '$[branch]'],
                    [actualParameterName: "skipTests", value: "false"]
                ]
                waitForPlaceholder true
            }
        }

        stage "Test", {
            gate "pre", {
                condition '$[/javascript]true'
            }
            task "Integration Tests", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "IntegrationTest"
                errorHandling "stopOnError"
            }
            task "Security Scan", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "SecurityScan"
                parallel      true
            }
        }

        stage "Deploy", {
            gate "pre", {
                task "Approve Deploy", {
                    taskType             "APPROVAL"
                    approver             "release-manager"
                    notificationTemplate "ApprovalRequest"
                }
            }
            task "Deploy to Target", {
                taskType        "DEPLOYER"
                deployerRunType "serial"
            }
        }
    }
}
```

### API Call Style

```groovy
def run = runPipeline(
    projectName:  "WebApp",
    pipelineName: "CI-CD",
    actualParameter: [
        [actualParameterName: "branch",    value: "release/3.0"],
        [actualParameterName: "targetEnv", value: "staging"]
    ]
)
println "Flow runtime ID: ${run.flowRuntime.flowRuntimeId}"

abortPipelineRun(
    flowRuntimeId: run.flowRuntime.flowRuntimeId,
    force:         false
)

def status = getPipelineRun(
    flowRuntimeId: run.flowRuntime.flowRuntimeId
)
println "Status: ${status.flowRuntime.status}"
```

---

## task

### taskType Options

| taskType | Description |
|---|---|
| `PROCEDURE` | Run a CD/RO procedure |
| `APPROVAL` | Manual approval gate |
| `DEPLOYER` | Deploy an application |
| `MANUAL` | Manual task with instructions |
| `PIPELINE` | Invoke another pipeline |
| `PLUGIN` | Call a plugin step directly |

### PROCEDURE task

```groovy
task "Run Build", {
    taskType      "PROCEDURE"
    projectName   "WebApp"
    procedureName "Build"
    actualParameter [
        [actualParameterName: "branch", value: '$[branch]']
    ]
    errorHandling "stopOnError"
    retries       2
    retryInterval 60
}
```

### APPROVAL task

```groovy
task "Production Approval", {
    taskType             "APPROVAL"
    approver             "release-manager,security-team"
    notificationTemplate "ApprovalRequest"
    approvalType         "required"
    instruction          "Review change log at https://wiki.example.com/releases/3.0"
}
```

### MANUAL task

```groovy
task "Smoke Test Prod", {
    taskType    "MANUAL"
    instruction "Verify https://app.example.com/health returns 200."
    assignees   "qa-team"
}
```

### API Call Style — complete a manual task

```groovy
completeManualTask(
    projectName:   "WebApp",
    pipelineName:  "CI-CD",
    stageName:     "Deploy",
    taskName:      "Smoke Test Prod",
    flowRuntimeId: "flowruntime-000001",
    status:        "pass",
    notes:         "All checks green"
)
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `taskType` | String | Yes | Type of task |
| `projectName` | String | Cond. | Project for PROCEDURE/PIPELINE tasks |
| `procedureName` | String | Cond. | Procedure for PROCEDURE tasks |
| `errorHandling` | String | No | `stopOnError`, `continueOnError`, `stopStage` |
| `parallel` | Boolean | No | Run concurrently with adjacent parallel tasks |
| `condition` | String | No | JavaScript; task skipped if false |
| `retries` | Integer | No | Number of automatic retries on failure |
| `retryInterval` | Integer | No | Seconds between retries |

---

## release

### Closure Style

```groovy
project "WebApp", {
    release "Release-3.0", {
        description      "Q4 2025 major release"
        plannedStartDate "2025-11-01"
        plannedEndDate   "2025-12-15"

        pipeline "Release-Pipeline", {
            stage "Feature Freeze", {
                task "Code Freeze Announcement", {
                    taskType    "MANUAL"
                    instruction "Send code freeze notice to all teams."
                    assignees   "release-manager"
                }
            }

            stage "QA Validation", {
                task "Full Regression", {
                    taskType      "PROCEDURE"
                    projectName   "WebApp"
                    procedureName "FullRegression"
                }
            }

            stage "Production Deploy", {
                gate "pre", {
                    task "Release Approval", {
                        taskType     "APPROVAL"
                        approver     "cto@example.com"
                        approvalType "required"
                    }
                }
                task "Blue-Green Deploy", {
                    taskType      "PROCEDURE"
                    projectName   "WebApp"
                    procedureName "BlueGreenDeploy"
                    actualParameter [
                        [actualParameterName: "version", value: "3.0.0"]
                    ]
                }
            }
        }
    }
}
```

### API Call Style

```groovy
def rel = startRelease(
    projectName: "WebApp",
    releaseName: "Release-3.0"
)
println "Release run ID: ${rel.flowRuntime.flowRuntimeId}"

def status = getRelease(
    projectName: "WebApp",
    releaseName: "Release-3.0"
)
println "Status: ${status.release.status}"
```

---

## Full Nesting Example

```groovy
project "WebApp", {
    pipeline "Full-CI-CD", {
        description "Build → Test → Approve → Deploy"

        formalParameter "branch", {
            defaultValue "main"
            required     true
            type         "entry"
        }

        stage "Build", {
            colorCode "#0078d4"
            task "Compile", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "Build"
                actualParameter [
                    [actualParameterName: "branch", value: '$[branch]']
                ]
                errorHandling "stopOnError"
            }
        }

        stage "Test", {
            colorCode "#ffc107"
            task "Unit Tests", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "UnitTest"
            }
            task "Integration Tests", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "IntegrationTest"
                parallel      true
            }
        }

        stage "Deploy to Staging", {
            colorCode "#17a2b8"
            gate "pre", {
                condition '$[/javascript]true'
            }
            task "Deploy", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "Deploy"
                actualParameter [
                    [actualParameterName: "env", value: "staging"]
                ]
            }
            gate "post", {
                task "Health Check", {
                    taskType      "PROCEDURE"
                    projectName   "WebApp"
                    procedureName "HealthCheck"
                }
            }
        }

        stage "Production", {
            colorCode "#dc3545"
            gate "pre", {
                task "Approve Production", {
                    taskType     "APPROVAL"
                    approver     "release-manager"
                    approvalType "required"
                }
            }
            task "Deploy to Production", {
                taskType      "PROCEDURE"
                projectName   "WebApp"
                procedureName "Deploy"
                actualParameter [
                    [actualParameterName: "env", value: "production"]
                ]
            }
        }
    }
}
```