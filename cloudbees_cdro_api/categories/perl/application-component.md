# CloudBees CD/RO Perl API — Application, Component & Process Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Application Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createApplication` | Creates a new application | `projectName`, `applicationName` |
| `deleteApplication` | Deletes an application | `projectName`, `applicationName` |
| `getApplication` | Retrieves an application | `projectName`, `applicationName` |
| `deployApplication` | Deploys to a target environment | `projectName`, `applicationName` |
| `createApplicationTier` | Creates a tier within an application | `projectName`, `applicationName`, `applicationTierName` |
| `createSnapshot` | Captures component versions as a snapshot | `projectName`, `applicationName`, `snapshotName` |
| `createComponent` | Creates a deployable component | `projectName`, `componentName` |
| `createProcess` | Creates a deploy/undeploy process | `projectName`, `processName` |
| `createProcessStep` | Adds a step to a process | `projectName`, `processName`, `processStepName` |
| `runProcess` | Executes an application process | `projectName`, `applicationName`, `processName` |
| `copyComponent` | Copies a component | `projectName`, `componentName`, `newComponentName` |
| `createProcessDependency` | Creates ordering edge between steps | `projectName`, `processName`, `processStepName`, `targetProcessStepName` |

---

### `deployApplication`

```perl
my $result = $cmdr->deployApplication("Pet Store", "Shopping Cart", {
    environmentName => "Production",
    snapshotName    => "release-2.1.0",
    processName     => "Deploy",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $jobId = $result->findvalue('//jobId')->string_value();
$cmdr->waitForJob($jobId, 600);
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `createComponent` + `createProcess` + `createProcessStep`

```perl
# Artifact-based component
$cmdr->createComponent("Pet Store", "Cart Service", {
    applicationName     => "Shopping Cart",
    applicationTierName => "App Tier",
    pluginKey           => "EC-Artifact",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Deploy process
$cmdr->createProcess("Pet Store", "Deploy Cart Service", {
    applicationName => "Shopping Cart",
    componentName   => "Cart Service",
    processType     => "DEPLOY",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Steps
$cmdr->createProcessStep("Pet Store", "Deploy Cart Service", "Download Artifact", {
    applicationName => "Shopping Cart",
    componentName   => "Cart Service",
    processStepType => "plugin",
    pluginKey       => "EC-Artifact",
    actualParameter => [
        {actualParameterName => "artifactName",  value => "petstore:cart-service"},
        {actualParameterName => "retrieveToDir", value => "/opt/petstore/app"},
    ],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `createSnapshot`

```perl
$cmdr->createSnapshot("Pet Store", "Shopping Cart", "release-2.1.0", {
    description => "Q2 release candidate — build 1042",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```