# CloudBees CD/RO Perl API — Parameter, Plugin & Email Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Parameter Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createFormalParameter` | Creates a typed input parameter | `projectName`, `formalParameterName` |
| `deleteFormalParameter` | Deletes a formal parameter | `projectName`, `formalParameterName` |
| `getFormalParameter` | Retrieves a formal parameter | `projectName`, `formalParameterName` |
| `getFormalParameters` | Lists all formal parameters | `projectName` |
| `modifyFormalParameter` | Modifies a parameter | `projectName`, `formalParameterName` |
| `attachParameter` | Attaches a parameter to a step | `projectName`, `procedureName`, `stepName`, `formalParameterName` |
| `detachParameter` | Detaches a parameter from a step | `projectName`, `procedureName`, `stepName`, `formalParameterName` |
| `setOutputParameter` | Sets an output parameter on a job/step | `outputParameterName`, `value` |
| `getOutputParameter` | Retrieves an output parameter | `outputParameterName` |
| `createFormalOutputParameter` | Declares a formal output parameter | `projectName`, `formalOutputParameterName` |
| `getFormalOutputParameters` | Lists all formal output parameters | `projectName` |

---

### `createFormalParameter`

```perl
# Text parameter
$cmdr->createFormalParameter("Pet Store", "branchName", {
    procedureName => "Build and Package WAR",
    type          => "text",
    label         => "Git Branch",
    defaultValue  => "main",
    required      => "1",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Select parameter
$cmdr->createFormalParameter("Pet Store", "targetEnvironment", {
    procedureName => "Deploy to Environment",
    type          => "select",
    label         => "Target Environment",
    defaultValue  => "QA East",
    required      => "1",
    options       => [
        {value => "QA East",            label => "QA (East)"},
        {value => "Staging US East",    label => "Staging (US East)"},
        {value => "Production US East", label => "Production (US East)"},
    ],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `setOutputParameter` / `getOutputParameter`

```perl
# Inside a running step
$cmdr->setOutputParameter("publishedArtifact", "petstore:cart-service:2.1.0-build.112");
die $cmdr->getError()->{message} if $cmdr->getError();

# From outside the job (after completion)
my $result = $cmdr->getOutputParameter("publishedArtifact", {
    jobId => "4fa765dd-73f1-11e3-b67e-b0a420524153",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $artifact = $result->findvalue('//outputParameter/value')->string_value();
print "Artifact: $artifact\n";
```

---

## Plugin Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `installPlugin` | Installs a plugin from file or URL | `url` |
| `uninstallPlugin` | Uninstalls a plugin | `pluginName` |
| `promotePlugin` | Sets plugin as active version | `pluginName` |
| `deletePlugin` | Deletes plugin metadata | `pluginName` |
| `getPlugin` | Retrieves plugin metadata | `pluginName` |
| `getPlugins` | Lists all installed plugins | (none) |
| `createPluginConfiguration` | Creates a plugin configuration | `projectName`, `pluginConfigurationName` |
| `deletePluginConfiguration` | Deletes a plugin configuration | `projectName`, `pluginConfigurationName` |
| `getPluginConfiguration` | Retrieves a plugin configuration | `projectName`, `pluginConfigurationName` |
| `modifyPluginConfiguration` | Updates a plugin configuration | `projectName`, `pluginConfigurationName` |

---

### `installPlugin` + `promotePlugin`

```perl
my $result = $cmdr->installPlugin("/tmp/EC-Git-3.2.0.jar", {
    disableProjectTracking => 1,
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $pluginName = $result->findvalue('//plugin/pluginName')->string_value();
print "Installed: $pluginName\n";

$cmdr->promotePlugin("EC-Git-3.2.0");
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `createPluginConfiguration`

```perl
$cmdr->createPluginConfiguration("Pet Store", "github-petstore", {
    pluginKey   => "EC-Git",
    description => "GitHub connection for Pet Store",
    field       => [
        {fieldName => "authType",    value => "token"},
        {fieldName => "serverUrl",   value => "https://github.com"},
        {fieldName => "checkConnection", value => "true"},
    ],
    credential  => [
        {credentialName => "token_credential",
         credentialReferenceParameter => "token_credential"},
    ],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Email Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createEmailConfig` | Creates SMTP config | `configName` |
| `modifyEmailConfig` | Updates SMTP settings | `configName` |
| `createEmailNotifier` | Creates an event-driven notifier | `notifierName` |
| `modifyEmailNotifier` | Modifies a notifier | `notifierName` |
| `sendEmail` | Sends an email directly | (all named) |

---

### `createEmailConfig` + `createEmailNotifier`

```perl
$cmdr->createEmailConfig("smtp-primary", {
    mailHost         => "smtp.example.com",
    mailPort         => 587,
    mailFrom         => 'noreply@example.com',
    mailUser         => 'cdro-mailer@example.com',
    mailUserPassword => "Sm7pP\@ssword!",
    mailSsl          => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createEmailNotifier("Build Failure Alert", {
    projectName   => "Pet Store",
    procedureName => "Build and Package WAR",
    eventType     => "onFailure",
    destinations  => "build-team\@example.com",
    configName    => "smtp-primary",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `sendEmail`

```perl
$cmdr->sendEmail({
    configName => "smtp-primary",
    to         => ["dev-lead\@example.com"],
    subject    => "Build Result: $jobName",
    html       => "<h2>Build $outcome</h2>",
    attachment => ["/workspace/reports/test-results.html"],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```