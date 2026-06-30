# DSL Reference: Project, Procedure, Step

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/project
  https://docs.cloudbees.com/apidocs/dsl/latest/c/procedure
  https://docs.cloudbees.com/apidocs/dsl/latest/c/step
  https://docs.cloudbees.com/apidocs/dsl/latest/c/formalParameter
  https://docs.cloudbees.com/apidocs/dsl/latest/c/formalOutputParameter
-->

## Running DSL

```bash
ectool evalDsl --dslFile script.groovy          # Groovy file
ectool evalDsl --format yaml --dslFile s.yaml   # YAML file
ectool evalDsl "project 'MyApp'"                # inline string
```

**Tip — generate DSL for an existing object:**
```bash
ectool generateDsl --objectType project --objectName "WebApp"
ectool generateDsl --objectType procedure --objectName "Build" --projectName "WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `project` | Top-level container for procedures, pipelines, applications | `name` |
| `procedure` | Named set of steps that can be run as a job | `name` (inside project context) |
| `step` | Single execution unit within a procedure | `name`, `command` or `subprocedure` |
| `formalParameter` | Input parameter definition for a procedure or step | `name` |
| `formalOutputParameter` | Output parameter definition for a procedure | `name` |

---

## project

### Closure Style

```groovy
project "WebApp", {
    description "Web application CI/CD project"
    tracked true
    resourceName "local"

    procedure "Build", {
        description "Compile and package the application"
    }
}
```

### API Call Style

```groovy
createProject(
    projectName:   "WebApp",
    description:   "Web application CI/CD project",
    resourceName:  "local",
    tracked:       true
)

def proj = getProject(projectName: "WebApp")
println proj.project.description
```

### YAML DSL

```yaml
apiVersion: cloudbees.com/v2023.2
kind: project
metadata:
  name: WebApp
spec:
  description: Web application CI/CD project
  tracked: true
  resourceName: local
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Human-readable description |
| `tracked` | Boolean | No | Enable SCM tracking |
| `resourceName` | String | No | Default resource pool for steps |
| `workspaceName` | String | No | Default workspace |

---

## procedure

### Closure Style (nested inside project)

```groovy
project "WebApp", {
    procedure "Build", {
        description "Full build pipeline"
        jobNameTemplate '$[/server/jobCounter]'
        resourceName    "build-agents"
        timeLimitUnits  "minutes"
        timeLimit       "30"

        formalParameter "branch", {
            defaultValue   "main"
            description    "Git branch to build"
            required       true
            type           "entry"
        }

        formalParameter "skipTests", {
            defaultValue   "false"
            type           "checkbox"
            checkedValue   "true"
            uncheckedValue "false"
        }

        formalOutputParameter "artifactVersion", {
            description "Published artifact version"
        }

        step "Checkout", {
            command    "git clone -b $[branch] https://git.example.com/repo.git"
            shell      "/bin/bash"
            resourceName "build-agents"
        }

        step "Compile", {
            command    'mvn package -DskipTests=$[skipTests]'
            shell      "/bin/bash"
            workingDirectory "/workspace/repo"
        }
    }
}
```

### API Call Style

```groovy
createProcedure(
    projectName:   "WebApp",
    procedureName: "Build",
    description:   "Full build pipeline",
    resourceName:  "build-agents",
    timeLimit:     "30",
    timeLimitUnits: "minutes"
)

createStep(
    projectName:     "WebApp",
    procedureName:   "Build",
    stepName:        "Run Tests",
    command:         "mvn test",
    shell:           "/bin/bash",
    resourceName:    "build-agents"
)

def result = runProcedure(
    projectName:   "WebApp",
    procedureName: "Build",
    actualParameter: [
        [actualParameterName: "branch",    value: "release/2.0"],
        [actualParameterName: "skipTests", value: "false"]
    ],
    priority:   "normal",
    waitForJob: true
)
println "Job ID: ${result.jobId}  Status: ${result.outcome}"
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Procedure description |
| `resourceName` | String | No | Default resource for steps |
| `timeLimitUnits` | String | No | `minutes`, `hours`, `seconds` |
| `timeLimit` | String | No | Numeric limit for the chosen unit |
| `jobNameTemplate` | String | No | Template for generated job names |

---

## step

### Closure Style — Command Step

```groovy
step "Run Tests", {
    description     "Execute unit test suite"
    command         "mvn test -Dsurefire.failIfNoSpecifiedTests=false"
    shell           "/bin/bash"
    resourceName    "build-agents"
    workingDirectory "/workspace/repo"
    timeLimit       "20"
    timeLimitUnits  "minutes"
    alwaysRun       false
    parallel        false

    environmentVariable "MAVEN_OPTS", {
        value "-Xmx1g"
    }
}
```

### Closure Style — Subprocedure Step

```groovy
step "Deploy to Staging", {
    subprocedure "Deploy",
    projectName  "WebApp",
    actualParameter: [
        [actualParameterName: "env",     value: "staging"],
        [actualParameterName: "version", value: '$[artifactVersion]']
    ]
}
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `command` | String | Cond. | Shell command to run |
| `shell` | String | No | Shell interpreter path |
| `subprocedure` | String | Cond. | Call another procedure as a step |
| `resourceName` | String | No | Override resource for this step |
| `workingDirectory` | String | No | Working directory on agent |
| `timeLimit` | String | No | Step timeout value |
| `timeLimitUnits` | String | No | `seconds`, `minutes`, `hours` |
| `alwaysRun` | Boolean | No | Run even if prior steps failed |
| `parallel` | Boolean | No | Run concurrently with adjacent parallel steps |
| `condition` | String | No | Conditional expression; step skipped if false |
| `errorHandling` | String | No | `abortProcedure`, `abortProcedureNow`, `markProcedureError`, `ignore` |

---

## Full Nesting Example

```groovy
project "WebApp", {
    description "Main web application project"
    resourceName "build-agents"

    procedure "CI-Build", {
        description    "Checkout → compile → test → publish"
        resourceName   "build-agents"
        timeLimitUnits "minutes"
        timeLimit      "60"

        formalParameter "branch", {
            defaultValue "main"
            required     true
            type         "entry"
        }

        formalParameter "skipTests", {
            type           "checkbox"
            defaultValue   "false"
            checkedValue   "true"
            uncheckedValue "false"
        }

        formalOutputParameter "artifactVersion", {
            description "Published artifact version"
        }

        step "Checkout", {
            command  "git clone -b \$[branch] https://git.example.com/webapp.git /workspace/webapp"
            shell    "/bin/bash"
        }

        step "Compile", {
            command          "mvn package -DskipTests=\$[skipTests]"
            shell            "/bin/bash"
            workingDirectory "/workspace/webapp"
        }

        step "Run Tests", {
            command          "mvn verify"
            shell            "/bin/bash"
            workingDirectory "/workspace/webapp"
            condition        '$[/javascript]getProperty("/myJob/actualParameters/skipTests") !== "true"'
            timeLimitUnits   "minutes"
            timeLimit        "30"
        }

        step "Publish Artifact", {
            command '''
                VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
                mvn deploy -DskipTests
                ectool setOutputParameter artifactVersion "$VERSION"
            '''
            shell            "/bin/bash"
            workingDirectory "/workspace/webapp"
        }
    }
}
```