# CloudBees CD/RO Groovy API — Parameter, Plugin & Email

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`
> Example: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/createFormalParameter`

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
| `createFormalParameter` | Declares an input parameter on a procedure or pipeline | `projectName`, `formalParameterName` + object ref |
| `deleteFormalParameter` | Removes a formal parameter declaration | `projectName`, `formalParameterName` + object ref |
| `getFormalParameter` | Returns formal parameter details | `projectName`, `formalParameterName` + object ref |
| `modifyFormalParameter` | Updates formal parameter attributes | `projectName`, `formalParameterName` + object ref |
| `createActualParameter` | Sets a runtime parameter value for a step | `actualParameterName`, `value` + object ref |
| `getActualParameter` | Returns a runtime parameter value | `actualParameterName` + object ref |
| `modifyActualParameter` | Updates a runtime parameter value | `actualParameterName`, `value` + object ref |
| `setOutputParameter` | Writes an output parameter value from a step | `outputParameterName`, `value` |
| `getOutputParameter` | Reads an output parameter set by a step | `outputParameterName` + job step ref |
| `createFormalOutputParameter` | Declares an output parameter on a procedure | `projectName`, `procedureName`, `formalOutputParameterName` |
| `installPlugin` | Installs a plugin from a file or URL | `file` or `url` |
| `uninstallPlugin` | Uninstalls a plugin | `pluginName` |
| `promotePlugin` | Promotes a plugin to active (default) version | `pluginName` |
| `getPlugin` | Returns plugin details | `pluginName` |
| `getPlugins` | Lists all installed plugins | — |
| `createPluginConfiguration` | Creates a plugin configuration (connection profile) | `pluginName`, `configName` |
| `modifyPluginConfiguration` | Updates a plugin configuration | `pluginName`, `configName` |
| `createEmailConfig` | Creates an SMTP email server configuration | `emailConfigName` |
| `modifyEmailConfig` | Updates an email server configuration | `emailConfigName` |
| `createEmailNotifier` | Creates a notification rule on a procedure | `projectName`, `procedureName`, `notifierName` |
| `modifyEmailNotifier` | Updates a notification rule | `projectName`, `procedureName`, `notifierName` |
| `sendEmail` | Sends an ad-hoc email | `emailConfigName`, `toAddress`, `subject` |

---

## createFormalParameter

Declares an input parameter on a procedure, pipeline, or application process.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `formalParameterName` | String | **Required.** Parameter name |
| `procedureName` | String | Attach to a procedure |
| `pipelineName` | String | Attach to a pipeline |
| `applicationName` | String | Attach to an application process |
| `processName` | String | Attach to a specific process (with `applicationName`) |
| `description` | String | Parameter description |
| `defaultValue` | String | Value used when caller does not supply one |
| `required` | Boolean | Fail if not supplied (default false) |
| `type` | String | `text`, `textarea`, `checkbox`, `select`, `credential`, `entry` |
| `expansionDeferred` | Boolean | Defer property expansion until runtime |
| `optionsDSL` | String | Groovy expression that returns valid choices (for `select` type) |
| `label` | String | Human-friendly display label |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

[
    [name: "branch",      label: "Git Branch",  required: true,  defaultValue: "main",    type: "text"],
    [name: "environment", label: "Target Env",   required: true,  defaultValue: "staging", type: "select"],
    [name: "runTests",    label: "Run Tests",    required: false, defaultValue: "true",    type: "checkbox"],
    [name: "buildNumber", label: "Build Number", required: false, defaultValue: "",        type: "text"]
].each { p ->
    ef.createFormalParameter(
        projectName:         "PaymentService",
        procedureName:       "BuildAndTest",
        formalParameterName: p.name,
        label:               p.label,
        required:            p.required,
        defaultValue:        p.defaultValue,
        type:                p.type
    )
    println "Declared parameter: ${p.name}"
}

ef.createFormalParameter(
    projectName:         "PaymentService",
    pipelineName:        "Release-Pipeline",
    formalParameterName: "branch",
    label:               "Release Branch",
    required:            true,
    type:                "text"
)
```

---

## createFormalOutputParameter / setOutputParameter / getOutputParameter

Output parameters pass values produced by one procedure to its callers or to subsequent pipeline tasks.

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createFormalOutputParameter(
    projectName:               "PaymentService",
    procedureName:             "BuildAndTest",
    formalOutputParameterName: "artifactVersion",
    description:               "Published artifact version string"
)

ef.createFormalOutputParameter(
    projectName:               "PaymentService",
    procedureName:             "BuildAndTest",
    formalOutputParameterName: "gitCommit",
    description:               "Git commit SHA that was built"
)

// Inside a running job step — write the output
ef.setOutputParameter(
    outputParameterName: "artifactVersion",
    value:               "2.4.7-build.847"
)

ef.setOutputParameter(
    outputParameterName: "gitCommit",
    value:               "a3f8d912c4b7e5f2"
)

// Caller retrieves it after the job completes
def outParam = ef.getOutputParameter(
    outputParameterName: "artifactVersion",
    jobStepId:           "jobStep-00012345-6789-abcd-ef01-234567890abc"
)
println "Artifact version from build: ${outParam.outputParameter.value}"
```

---

## installPlugin / promotePlugin

Installs a plugin archive and promotes it to active use.

### Key Named Parameters — installPlugin

| Parameter | Type | Description |
|---|---|---|
| `file` | String | Local file path to the plugin `.jar` or `.zip` |
| `url` | String | Remote URL to download the plugin from |

### Key Named Parameters — promotePlugin

| Parameter | Type | Description |
|---|---|---|
| `pluginName` | String | **Required.** Plugin name to promote |
| `version` | String | Specific version to promote (default: latest installed) |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.installPlugin(file: "/tmp/EC-Kubernetes-2.5.0.jar")
println "Plugin installed"

ef.promotePlugin(pluginName: "EC-Kubernetes")
println "EC-Kubernetes promoted to active"

def plugins = ef.getPlugins()
plugins.plugin.each { plugin ->
    println "${plugin.pluginName}  version=${plugin.pluginVersion}  promoted=${plugin.promoted}"
}

def k8sPlugin = ef.getPlugin(pluginName: "EC-Kubernetes")
println "Plugin ID:   ${k8sPlugin.plugin.pluginId}"
println "Plugin key:  ${k8sPlugin.plugin.pluginKey}"
```

---

## createPluginConfiguration

Creates a named configuration (connection profile) for a plugin.

### Key Named Parameters

| Parameter | Type | Description |
|---|---|---|
| `pluginName` | String | **Required.** Plugin that owns this configuration |
| `configName` | String | **Required.** Unique configuration name |
| `description` | String | Configuration description |
| `*` | Various | Plugin-specific fields — see plugin docs |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createPluginConfiguration(
    pluginName:         "EC-Kubernetes",
    configName:         "prod-k8s-cluster",
    description:        "Production EKS cluster — us-east-1",
    kubernetesEndpoint: "https://eks.us-east-1.example.com",
    credentialName:     "eks-service-account",
    testConnection:     "true"
)

println "Plugin configuration created: prod-k8s-cluster"

ef.createPluginConfiguration(
    pluginName:    "EC-AWS",
    configName:    "aws-prod",
    description:   "AWS production account",
    region:        "us-east-1",
    credentialName: "aws-deployer"
)
```

---

## createEmailConfig / sendEmail / createEmailNotifier

### Key Named Parameters — createEmailConfig

| Parameter | Type | Description |
|---|---|---|
| `emailConfigName` | String | **Required.** Configuration name |
| `mailServer` | String | SMTP server hostname |
| `mailPort` | Integer | SMTP port (default 25) |
| `senderAddress` | String | From address |
| `userName` | String | SMTP auth username |
| `password` | String | SMTP auth password |
| `useSsl` | Boolean | Use SSL/TLS |

### Key Named Parameters — createEmailNotifier

| Parameter | Type | Description |
|---|---|---|
| `projectName` | String | **Required.** Parent project |
| `procedureName` | String | **Required.** Procedure to attach the notifier to |
| `notifierName` | String | **Required.** Unique notifier name |
| `emailConfigName` | String | SMTP configuration to use |
| `destinations` | String | Comma-separated recipient addresses |
| `eventType` | String | `onSuccess`, `onFailure`, `onCompletion`, `onStep` |
| `condition` | String | Extra condition expression |
| `template` | String | Email body template with property references |

### Example

```groovy
import com.electriccloud.client.groovy.ElectricFlow

ElectricFlow ef = new ElectricFlow()

ef.createEmailConfig(
    emailConfigName: "corporate-smtp",
    mailServer:      "smtp.example.com",
    mailPort:        587,
    senderAddress:   "cdro-noreply@example.com",
    userName:        System.getenv("SMTP_USER"),
    password:        System.getenv("SMTP_PASSWORD"),
    useSsl:          true
)

ef.createEmailNotifier(
    projectName:     "PaymentService",
    procedureName:   "BuildAndTest",
    notifierName:    "build-failure-alert",
    emailConfigName: "corporate-smtp",
    destinations:    "dev-team@example.com,ci-alerts@example.com",
    eventType:       "onFailure",
    template:        "Build FAILED: \$[/myJob/jobName] on branch \$[/myJob/branch]\\nJob ID: \$[/myJob/jobId]"
)

ef.createEmailNotifier(
    projectName:     "PaymentService",
    procedureName:   "BuildAndTest",
    notifierName:    "build-success-alert",
    emailConfigName: "corporate-smtp",
    destinations:    "release-managers@example.com",
    eventType:       "onSuccess",
    template:        "Build SUCCEEDED: \$[/myJob/jobName]\\nArtifact: \$[/myJob/artifactVersion]"
)

ef.sendEmail(
    emailConfigName: "corporate-smtp",
    toAddress:       "ops-team@example.com",
    subject:         "Production deployment initiated — payment-service \$[/myJob/buildVersion]",
    body:            "Deployment to production started at \$[/timestamp] by \$[/myJob/launchedByUser]."
)

println "Email configuration and notifiers set up"
```