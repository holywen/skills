# DSL Reference: Application, Component, Process, ProcessStep, ApplicationTier

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/application
  https://docs.cloudbees.com/apidocs/dsl/latest/c/component
  https://docs.cloudbees.com/apidocs/dsl/latest/c/process
  https://docs.cloudbees.com/apidocs/dsl/latest/c/processStep
  https://docs.cloudbees.com/apidocs/dsl/latest/c/applicationTier
  https://docs.cloudbees.com/apidocs/dsl/latest/c/deployerApplication
-->

## Running DSL

```bash
ectool evalDsl --dslFile application.groovy
ectool generateDsl --objectType application --objectName "WebApp" --projectName "WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `application` | Deployable unit composed of components and processes | `name` |
| `applicationTier` | Logical grouping of components within an application | `name` |
| `component` | Individual deployable artifact (JAR, WAR, Docker image, etc.) | `name` |
| `process` | Ordered sequence of process steps for deploy/undeploy/config | `name`, `processType` |
| `processStep` | Single step within a process (plugin call, procedure, etc.) | `name`, `processStepType` |
| `deployerApplication` | Reference binding an application to a pipeline deployer task | `name` |

---

## application

### Closure Style

```groovy
project "WebApp", {
    application "WebApp", {
        description "Three-tier web application"

        applicationTier "Web", {
            component "Frontend", projectName: "WebApp", {
                description "React SPA served via Nginx"
            }
        }

        applicationTier "App", {
            component "Backend", projectName: "WebApp", {
                description "Spring Boot REST API"
            }
        }

        applicationTier "Data", {
            component "Database", projectName: "WebApp", {
                description "PostgreSQL schema migrations"
            }
        }

        process "Deploy", {
            processType "DEPLOY"
            description "Full application deploy"

            processStep "Deploy Frontend", {
                processStepType "COMPONENT"
                componentName   "Frontend"
                applicationTierName "Web"
                subprocedure    "Deploy"
                processName     "Deploy"
            }

            processStep "Deploy Backend", {
                processStepType "COMPONENT"
                componentName   "Backend"
                applicationTierName "App"
                subprocedure    "Deploy"
                processName     "Deploy"
                dependsOn       "Deploy Frontend"
            }

            processStep "Run DB Migrations", {
                processStepType "COMPONENT"
                componentName   "Database"
                applicationTierName "Data"
                subprocedure    "Deploy"
                processName     "Deploy"
                dependsOn       "Deploy Frontend"
            }
        }

        process "Undeploy", {
            processType "UNDEPLOY"
            processStep "Stop Backend", {
                processStepType "COMPONENT"
                componentName   "Backend"
                applicationTierName "App"
                processName     "Undeploy"
            }
        }
    }
}
```

### API Call Style

```groovy
createApplication(
    projectName:     "WebApp",
    applicationName: "WebApp",
    description:     "Three-tier web application"
)

def result = deployApplication(
    projectName:     "WebApp",
    applicationName: "WebApp",
    environmentName: "staging",
    processName:     "Deploy",
    actualParameter: [
        [actualParameterName: "version", value: "3.0.1"]
    ]
)
println "Deploy job: ${result.jobId}"

def run = runProcess(
    projectName:     "WebApp",
    applicationName: "WebApp",
    processName:     "Deploy",
    environmentName: "staging"
)
println "Process job: ${run.jobId}"
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Application description |
| `applicationTier` | Block | No | Logical tier containing components |

---

## component

### Closure Style

```groovy
project "WebApp", {
    component "Backend", {
        description   "Spring Boot REST API"
        componentType "artifact"

        process "Deploy", {
            processType "DEPLOY"
            description "Retrieve artifact and restart service"

            processStep "Retrieve Artifact", {
                processStepType "PLUGIN"
                subprocedure    "Retrieve"
                projectName     "/plugins/EC-Artifact/project"
                actualParameter [
                    [actualParameterName: "artifactName",        value: "com.example:backend"],
                    [actualParameterName: "artifactVersion",     value: '$[/myJob/artifactVersion]'],
                    [actualParameterName: "retrieveToDirectory", value: "/opt/deploy/backend"]
                ]
            }

            processStep "Stop Service", {
                processStepType "COMMAND"
                command         "systemctl stop backend || true"
                shell           "/bin/bash"
                resourceName    '$[/myEnvironment/appServer]'
            }

            processStep "Start Service", {
                processStepType "COMMAND"
                command         "systemctl start backend"
                shell           "/bin/bash"
                resourceName    '$[/myEnvironment/appServer]'
                dependsOn       "Stop Service"
            }
        }

        process "Undeploy", {
            processType "UNDEPLOY"
            processStep "Stop and Remove", {
                processStepType "COMMAND"
                command         "systemctl stop backend && rm -rf /opt/deploy/backend"
                shell           "/bin/bash"
            }
        }
    }
}
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Component description |
| `componentType` | String | No | `artifact`, `source`, `other` |
| `pluginKey` | String | No | Plugin key for artifact retrieval |

---

## processStep

### processStepType Options

| processStepType | Description |
|---|---|
| `COMMAND` | Run a shell command on a resource |
| `PLUGIN` | Call a plugin procedure |
| `PROCEDURE` | Call a CD/RO procedure |
| `COMPONENT` | Invoke another component's process (app-level only) |
| `MANUAL` | Manual step requiring human action |

### Closure Style — COMMAND step

```groovy
processStep "Health Check", {
    processStepType  "COMMAND"
    command          "curl -f http://localhost:8080/health || exit 1"
    shell            "/bin/bash"
    resourceName     '$[/myEnvironment/appServer]'
    timeLimit        "2"
    timeLimitUnits   "minutes"
    errorHandling    "failProcedure"
    dependsOn        "Deploy Artifact"
}
```

### Closure Style — PLUGIN step

```groovy
processStep "Deploy to Kubernetes", {
    processStepType "PLUGIN"
    subprocedure    "Deploy"
    projectName     "/plugins/EC-Kubernetes/project"
    actualParameter [
        [actualParameterName: "configName",     value: "k8s-prod"],
        [actualParameterName: "namespace",      value: "webapp"],
        [actualParameterName: "deploymentName", value: "backend"],
        [actualParameterName: "imageName",      value: "registry.example.com/backend:\$[version]"]
    ]
}
```

### Closure Style — with dependsOn (sequencing)

```groovy
processStep "Step A", { processStepType "COMMAND"; command "echo A"; shell "/bin/bash" }
processStep "Step B", { processStepType "COMMAND"; command "echo B"; shell "/bin/bash"; dependsOn "Step A" }
processStep "Step C", { processStepType "COMMAND"; command "echo C"; shell "/bin/bash"; dependsOn "Step A" }
processStep "Step D", { processStepType "COMMAND"; command "echo D"; shell "/bin/bash"; dependsOn "Step B,Step C" }
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `processStepType` | String | Yes | Type of step |
| `command` | String | Cond. | Shell command for COMMAND steps |
| `shell` | String | No | Shell path, default `/bin/sh` |
| `subprocedure` | String | Cond. | Plugin procedure for PLUGIN steps |
| `resourceName` | String | No | Override resource for this step |
| `dependsOn` | String | No | Comma-separated list of step names to complete first |
| `errorHandling` | String | No | `failProcedure`, `abortProcedure`, `ignore` |
| `parallel` | Boolean | No | Run in parallel with other parallel steps |

---

## Full Nesting Example

```groovy
project "WebApp", {
    application "WebApp", {
        description "Full three-tier web application"

        applicationTier "Web", {
            component "Frontend", projectName: "WebApp", {
                description   "React SPA"
                componentType "artifact"

                process "Deploy", {
                    processType "DEPLOY"
                    processStep "Retrieve SPA Bundle", {
                        processStepType "PLUGIN"
                        subprocedure    "Retrieve"
                        projectName     "/plugins/EC-Artifact/project"
                        actualParameter [
                            [actualParameterName: "artifactName",        value: "com.example:frontend"],
                            [actualParameterName: "artifactVersion",     value: '$[/myJob/artifactVersion]'],
                            [actualParameterName: "retrieveToDirectory", value: "/var/www/html"]
                        ]
                    }
                    processStep "Reload Nginx", {
                        processStepType "COMMAND"
                        command         "systemctl reload nginx"
                        shell           "/bin/bash"
                        dependsOn       "Retrieve SPA Bundle"
                    }
                }
            }
        }

        applicationTier "App", {
            component "Backend", projectName: "WebApp", {
                description   "Spring Boot REST API"
                componentType "artifact"

                process "Deploy", {
                    processType "DEPLOY"
                    processStep "Retrieve JAR", {
                        processStepType "PLUGIN"
                        subprocedure    "Retrieve"
                        projectName     "/plugins/EC-Artifact/project"
                        actualParameter [
                            [actualParameterName: "artifactName",        value: "com.example:backend"],
                            [actualParameterName: "artifactVersion",     value: '$[/myJob/artifactVersion]'],
                            [actualParameterName: "retrieveToDirectory", value: "/opt/deploy/backend"]
                        ]
                    }
                    processStep "Restart Service", {
                        processStepType "COMMAND"
                        command         "systemctl restart backend"
                        shell           "/bin/bash"
                        dependsOn       "Retrieve JAR"
                    }
                    processStep "Health Check", {
                        processStepType "COMMAND"
                        command         "curl -sf http://localhost:8080/health"
                        shell           "/bin/bash"
                        dependsOn       "Restart Service"
                    }
                }
            }
        }

        process "Deploy All", {
            processType "DEPLOY"
            processStep "Deploy Frontend", {
                processStepType     "COMPONENT"
                componentName       "Frontend"
                applicationTierName "Web"
                processName         "Deploy"
            }
            processStep "Deploy Backend", {
                processStepType     "COMPONENT"
                componentName       "Backend"
                applicationTierName "App"
                processName         "Deploy"
                dependsOn           "Deploy Frontend"
            }
        }
    }
}
```