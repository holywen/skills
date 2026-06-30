# CloudBees CD/RO Perl API — Workspace, Workflow, Catalog & Server

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Workspace, Gateway & Zone Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createWorkspace` | Creates a workspace | `workspaceName` |
| `deleteWorkspace` | Deletes a workspace | `workspaceName` |
| `modifyWorkspace` | Updates workspace paths | `workspaceName` |
| `createGateway` | Creates a cross-zone gateway | `gatewayName` |
| `modifyGateway` | Updates gateway resources | `gatewayName` |
| `createZone` | Creates a zone | `zoneName` |
| `modifyZone` | Updates a zone | `zoneName` |

---

### `createWorkspace`

```perl
$cmdr->createWorkspace("builds", {
    description    => "Primary build workspace",
    agentUnixPath  => "/mnt/builds",
    agentDrivePath => "c:/builds",
    agentUncPath   => "//fileserver/builds",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Workflow Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createWorkflowDefinition` | Creates a workflow definition | `projectName`, `workflowDefinitionName` |
| `modifyWorkflowDefinition` | Updates a workflow definition | `projectName`, `workflowDefinitionName` |
| `createStateDefinition` | Creates a state | `projectName`, `workflowDefinitionName`, `stateDefinitionName` |
| `createTransitionDefinition` | Creates a transition | `projectName`, `workflowDefinitionName`, `stateDefinitionName`, `transitionDefinitionName`, `targetState` |
| `runWorkflow` | Launches a workflow instance | `projectName`, `workflowDefinitionName` |
| `completeWorkflow` | Marks workflow as completed | `projectName`, `workflowName` |
| `transitionWorkflow` | Fires a manual transition | `projectName`, `workflowName`, `stateName`, `transitionName` |
| `getWorkflow` | Gets a workflow instance | `projectName`, `workflowName` |
| `getWorkflows` | Lists workflow instances | `projectName` |
| `createTrigger` | Creates a SCM/webhook trigger | `projectName`, `triggerName` |
| `runTrigger` | Manually fires a trigger | (all named) |
| `createScmSync` | Creates a Git SCM sync | `projectName`, `scmSyncName` |
| `runScmSync` | Executes an SCM sync | `projectName`, `scmSyncName` |

---

### `createWorkflowDefinition` + states + transitions

```perl
$cmdr->createWorkflowDefinition("Pet Store", "Build-Test-Deploy", {
    description => "End-to-end build, test, and deploy workflow",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createStateDefinition("Pet Store", "Build-Test-Deploy", "Build", {
    procedureName => "Build and Package WAR",
    startable     => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createStateDefinition("Pet Store", "Build-Test-Deploy", "Test", {
    procedureName => "Run Integration Tests",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createTransitionDefinition("Pet Store", "Build-Test-Deploy",
    "Build", "build-to-test", "Test", {
        trigger    => "onCompletion",
        condition  => '$[/myJob/outcome] == "success"',
    });
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `runWorkflow` / `transitionWorkflow`

```perl
my $result = $cmdr->runWorkflow("Pet Store", "Build-Test-Deploy", {
    startingState => "Build",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $workflowName = $result->findvalue('//workflow/workflowName')->string_value();

$cmdr->transitionWorkflow("Pet Store", $workflowName, "Test", "test-to-deploy");
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `createTrigger`

```perl
$cmdr->createTrigger("Pet Store", "github-push-main", {
    pipelineName    => "Release Pipeline",
    triggerType     => "webhook",
    pluginKey       => "EC-Github",
    pluginParameter => [
        {pluginParameterName => "repositories", value => "petstore-org/shopping-cart"},
        {pluginParameterName => "eventType",    value => "push"},
        {pluginParameterName => "branchFilter", value => "main"},
    ],
    quietTime => 30,
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Catalog Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createCatalog` | Creates a self-service catalog | `projectName`, `catalogName` |
| `createCatalogItem` | Creates a catalog item | `projectName`, `catalogName`, `catalogItemName` |
| `deleteCatalogItem` | Deletes a catalog item | `projectName`, `catalogName`, `catalogItemName` |
| `getCatalogItem` | Retrieves a catalog item | `projectName`, `catalogName`, `catalogItemName` |
| `getCatalogItems` | Lists catalog items | `projectName`, `catalogName` |
| `runCatalogItem` | Executes a catalog item | `projectName`, `catalogName`, `catalogItemName` |

---

## Server Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `getServerStatus` | Retrieves server status | (none) |
| `getVersions` | Retrieves server version | (none) |
| `evalDsl` | Evaluates a DSL script | `dsl` or `dslFile` (named) |
| `findObjects` | Searches for objects by type and filter (Perl only) | `objectType` (named) |
| `export` | Exports server data to XML | `fileName` |
| `import` | Imports from XML | `fileName` |
| `generateDsl` | Generates DSL for an object | `path` (named) |
| `clone` | Clones a CD/RO entity | (all named) |
| `sendReportingData` | Sends data to DevOps Insight | (all named) |
| `tagObject` | Tags an object | (all named) |
| `revert` | Reverts to a prior revision | (all named) |

---

### `findObjects` (Perl only)

```perl
# Find all failed jobs in Pet Store in last 24 hours
my $result = $cmdr->findObjects('job', {
    filter => [{
        operator => 'and',
        filter   => [
            {propertyName => "projectName", operator => "equals",      operand1 => "Pet Store"},
            {propertyName => "outcome",     operator => "equals",      operand1 => "error"},
            {propertyName => "start",       operator => "greaterThan", operand1 => time() - 86400},
        ],
    }],
    sortKey    => "start",
    sortOrder  => "descending",
    maxResults => 50,
});
die $cmdr->getError()->{message} if $cmdr->getError();
my @jobIds = map { $_->string_value() } $result->findnodes('//job/jobId');
print "Failed jobs: " . scalar(@jobIds) . "\n";
```

---

### `evalDsl`

```perl
# Inline DSL
$cmdr->evalDsl("project 'Microservices Platform'");
die $cmdr->getError()->{message} if $cmdr->getError();

# From file
$cmdr->evalDsl({
    dslFile    => "/workspace/definitions/release-pipeline.groovy",
    overwrite  => "true",
    projectName => "Pet Store",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `generateDsl` + `export`

```perl
# Generate DSL for a pipeline
my $result = $cmdr->generateDsl({
    path             => "/projects/Pet Store/pipelines/Release Pipeline",
    suppressDefaults => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $dsl = $result->findvalue('//dsl')->string_value();
open(my $fh, '>', "/workspace/definitions/release-pipeline.groovy") or die $!;
print $fh $dsl;
close $fh;

# Export project
$cmdr->export("/backup/pet-store.xml", {
    path        => "/projects[Pet Store]",
    relocatable => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `sendReportingData`

```perl
use JSON;
my $payload = encode_json({
    applicationName  => "Shopping Cart v2",
    environmentName  => "Production US East",
    deployedVersion  => "2.1.0-build.112",
    deploymentStatus => "SUCCESS",
});
$cmdr->sendReportingData({
    reportObjectTypeName => "DEPLOYMENT",
    payload             => $payload,
    projectName         => "Pet Store",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```