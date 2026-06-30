# DSL Reference: Parameters, Plugin, Email

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/formalParameter
  https://docs.cloudbees.com/apidocs/dsl/latest/c/actualParameter
  https://docs.cloudbees.com/apidocs/dsl/latest/c/outputParameter
  https://docs.cloudbees.com/apidocs/dsl/latest/c/plugin
  https://docs.cloudbees.com/apidocs/dsl/latest/c/pluginConfiguration
  https://docs.cloudbees.com/apidocs/dsl/latest/c/emailConfig
  https://docs.cloudbees.com/apidocs/dsl/latest/c/emailNotifier
-->

## Running DSL

```bash
ectool evalDsl --dslFile params.groovy
ectool generateDsl --objectType plugin --objectName "EC-Git"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `formalParameter` | Input parameter definition on a procedure, pipeline, or process | `name` |
| `actualParameter` | Runtime value passed when invoking a procedure/step | `actualParameterName`, `value` |
| `formalOutputParameter` | Output parameter definition declared on a procedure | `name` |
| `plugin` | Installed CD/RO plugin providing additional step types | `name` |
| `pluginConfiguration` | Named plugin instance configuration (credentials, endpoints) | `name` |
| `emailConfig` | Server-level SMTP configuration | (server-level object) |
| `emailNotifier` | Notification rule attached to a procedure or job | `notifierName` |

---

## formalParameter

### Closure Style — all parameter types

```groovy
procedure "Deploy", {

    formalParameter "version", {
        description  "Artifact version to deploy (e.g. 3.0.1)"
        required     true
        type         "entry"
        defaultValue "LATEST"
    }

    formalParameter "extraConfig", {
        description  "Additional JSON configuration block"
        required     false
        type         "textarea"
        defaultValue "{}"
    }

    formalParameter "environment", {
        description  "Target deployment environment"
        required     true
        type         "select"
        options {
            option "Development", { value "dev" }
            option "Staging",     { value "stg" }
            option "Production",  { value "prd" }
        }
        defaultValue "dev"
    }

    formalParameter "dryRun", {
        description    "Simulate deployment without making changes"
        type           "checkbox"
        defaultValue   "false"
        checkedValue   "true"
        uncheckedValue "false"
    }

    formalParameter "deployCredential", {
        description "Credential used for deployment"
        type        "credential"
        required    true
    }
}
```

### API Call Style

```groovy
createFormalParameter(
    projectName:         "WebApp",
    procedureName:       "Deploy",
    formalParameterName: "version",
    required:            true,
    type:                "entry",
    defaultValue:        "LATEST",
    description:         "Artifact version to deploy"
)

modifyFormalParameter(
    projectName:         "WebApp",
    procedureName:       "Deploy",
    formalParameterName: "version",
    defaultValue:        "3.0.0"
)

def params = getFormalParameters(
    projectName:   "WebApp",
    procedureName: "Deploy"
)
params.formalParameter.each { p ->
    println "${p.formalParameterName}: type=${p.type} required=${p.required}"
}
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `type` | String | No | `entry`, `textarea`, `select`, `checkbox`, `credential`, `project`, `resource` |
| `defaultValue` | String | No | Pre-filled default shown in launch dialog |
| `required` | Boolean | No | User must supply a value |
| `description` | String | No | Help text displayed next to parameter |
| `checkedValue` | String | No | Value when checkbox is checked |
| `uncheckedValue` | String | No | Value when checkbox is unchecked |
| `expansionDeferred` | Boolean | No | Defer `$[...]` expansion until runtime |

---

## actualParameter

### In runProcedure

```groovy
def result = runProcedure(
    projectName:   "WebApp",
    procedureName: "Deploy",
    actualParameter: [
        [actualParameterName: "version",     value: "3.0.1"],
        [actualParameterName: "environment", value: "stg"],
        [actualParameterName: "dryRun",      value: "false"]
    ],
    waitForJob: true
)
println "Outcome: ${result.outcome}"
```

### In a subprocedure step

```groovy
step "Invoke Deploy", {
    subprocedure "Deploy"
    projectName  "WebApp"
    actualParameter [
        [actualParameterName: "version",     value: '$[/myJob/artifactVersion]'],
        [actualParameterName: "environment", value: "stg"]
    ]
}
```

### In a pipeline task

```groovy
task "Deploy to Staging", {
    taskType      "PROCEDURE"
    projectName   "WebApp"
    procedureName "Deploy"
    actualParameter [
        [actualParameterName: "version",     value: '$[/myPipelineRuntime/parameters/version]'],
        [actualParameterName: "environment", value: "stg"]
    ]
}
```

---

## formalOutputParameter

### Declare on procedure

```groovy
procedure "Build", {
    formalOutputParameter "artifactVersion", {
        description "Published artifact version"
    }
    formalOutputParameter "buildNumber", {
        description "Numeric build counter"
    }
}
```

### Set during step execution

```groovy
step "Set Output Params", {
    command '''
        VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
        ectool setOutputParameter artifactVersion "$VERSION"
        ectool setOutputParameter buildNumber     "42"
    '''
    shell "/bin/bash"
}
```

### Read output from calling context

```groovy
def job = runProcedure(
    projectName:   "WebApp",
    procedureName: "Build",
    waitForJob:    true
)

def version = getProperty(
    propertyName: "artifactVersion",
    jobId:        job.jobId
).property.value

println "Built version: ${version}"
```

---

## plugin

### API Call Style — install and promote

```groovy
installPlugin(
    file: "/opt/plugins/EC-Git-2.7.0.jar"
)

promotePlugin(
    pluginName: "EC-Git-2.7.0"
)

def plugin = getPlugin(pluginName: "EC-Git")
println "Version: ${plugin.plugin.pluginVersion}"
println "Promoted: ${plugin.plugin.promoted}"

def plugins = getPlugins()
plugins.plugin.each { p ->
    println "${p.pluginName}  promoted=${p.promoted}"
}
```

---

## emailConfig

### Closure Style (server-level)

```groovy
emailConfig {
    mailServer  "smtp.example.com"
    mailPort    587
    mailFrom    "cdro-noreply@example.com"
    userName    "svc-cdro-email"
    password    "PLACEHOLDER"
    useSsl      false
    useTls      true
    mailTo      "cdro-alerts@example.com"
}
```

### API Call Style

```groovy
modifyEmailConfig(
    mailServer: "smtp.example.com",
    mailPort:   587,
    mailFrom:   "cdro-noreply@example.com",
    userName:   "svc-cdro-email",
    password:   "PLACEHOLDER",
    useTls:     true
)
```

---

## emailNotifier

### Closure Style

```groovy
project "WebApp", {
    procedure "Build", {
        emailNotifier "On-Failure", {
            condition    "error"
            destinations "build-alerts@example.com,ops@example.com"
            subject      "BUILD FAILED: \$[/myJob/jobName] on branch \$[branch]"
            body         "Job: \$[/myJob/jobName]\nOutcome: \$[/myJob/outcome]\nLog: \$[/myJob/logURL]"
            notifyOnLastSuccess true
        }

        emailNotifier "Always-Notify-Leads", {
            condition    "always"
            destinations "release-managers@example.com"
            subject      "Build completed: \$[/myJob/jobName] — \$[/myJob/outcome]"
            body         "See job log: \$[/myJob/logURL]"
        }
    }
}
```

### API Call Style — sendEmail

```groovy
sendEmail(
    destinations: "ops-team@example.com",
    subject:      "Deploy completed: ${version} → production",
    body:         "Deployment succeeded.\nVersion: ${version}\nEnvironment: production"
)
```

### Key Properties — emailNotifier

| Property | Type | Required | Description |
|---|---|---|---|
| `condition` | String | Yes | `always`, `error`, `warning`, `success` |
| `destinations` | String | Yes | Comma-separated email addresses |
| `subject` | String | No | Email subject line (supports `$[...]` expansion) |
| `body` | String | No | Email body (supports `$[...]` expansion) |
| `notifyOnLastSuccess` | Boolean | No | Also notify when job recovers after failure |

---

## Full Example — procedure with all parameter types and email notification

```groovy
project "WebApp", {
    procedure "Full-Deploy", {
        description "Deploy with full parameter set and notifications"

        formalParameter "version", {
            required     true
            type         "entry"
            description  "Artifact version (e.g. 3.0.1)"
        }

        formalParameter "environment", {
            required     true
            type         "select"
            options {
                option "Staging",    { value "stg" }
                option "Production", { value "prd" }
            }
            defaultValue "stg"
        }

        formalParameter "dryRun", {
            type           "checkbox"
            defaultValue   "false"
            checkedValue   "true"
            uncheckedValue "false"
            description    "Simulate only — no real deploy"
        }

        formalOutputParameter "deployedUrl", {
            description "URL of the deployed application"
        }

        step "Deploy", {
            command '''
                if [ "$[dryRun]" = "true" ]; then
                    echo "DRY RUN: would deploy $[version] to $[environment]"
                    ectool setOutputParameter deployedUrl "https://$[environment].example.com (dry run)"
                else
                    ./deploy.sh --version $[version] --env $[environment]
                    ectool setOutputParameter deployedUrl "https://$[environment].example.com"
                fi
            '''
            shell "/bin/bash"
        }

        emailNotifier "Deploy-Failure-Alert", {
            condition    "error"
            destinations "ops-team@example.com,release-managers@example.com"
            subject      "DEPLOY FAILED: \$[/myJob/jobName] v\$[version] → \$[environment]"
            body         "Job log: \$[/myJob/logURL]"
            notifyOnLastSuccess true
        }

        emailNotifier "Production-Deploy-Notice", {
            condition    "success"
            destinations "stakeholders@example.com"
            subject      "Production deploy succeeded: v\$[version]"
            body         "Application available at: \$[/myJob/outputParameters/deployedUrl]"
        }
    }
}
```