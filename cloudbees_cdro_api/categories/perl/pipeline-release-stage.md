# CloudBees CD/RO Perl API — Pipeline, Release & Stage Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Pipeline Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createPipeline` | Creates a new pipeline | `projectName`, `pipelineName` |
| `runPipeline` | Starts a pipeline run | `projectName`, `pipelineName` |
| `abortPipelineRun` | Aborts an active pipeline run | `flowRuntimeId` (named) |
| `pausePipelineRun` | Pauses a pipeline run | `flowRuntimeId` (named) |
| `resumePipelineRun` | Resumes a paused pipeline run | `flowRuntimeId` (named) |
| `waitForFlowRuntime` | Waits for pipeline run to complete (Perl only) | `flowRuntimeId` (named) |
| `restartPipelineRun` | Restarts from a specified stage | (named) |

---

### `runPipeline` + `waitForFlowRuntime`

```perl
my $result = $cmdr->runPipeline("Pet Store", "Release Pipeline", {
    actualParameter => [
        {actualParameterName => "releaseVersion", value => "2.1.0"},
    ],
    priority => "high",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $frid = $result->findvalue('//flowRuntime/flowRuntimeId')->string_value();

$cmdr->waitForFlowRuntime($frid, {timeout => 1800});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Release Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createRelease` | Creates a new release | `projectName`, `releaseName` |
| `startRelease` | Starts a release run | `projectName`, `releaseName` |
| `completeRelease` | Marks a release as completed | `projectName`, `releaseName` |
| `attachPipelineRun` | Attaches a pipeline run to a release | named args |
| `detachPipelineRun` | Detaches a pipeline run from a release | named args |
| `getReleaseTimelineDetails` | Returns timing data for a release | `projectName`, `releaseName` |

---

### `createRelease` + `startRelease`

```perl
$cmdr->createRelease("Pet Store", "v2.1.0", {
    description      => "Q2 feature release",
    pipelineName     => "Release Pipeline",
    plannedStartDate => "2026-07-01",
    plannedEndDate   => "2026-07-15",
});
die $cmdr->getError()->{message} if $cmdr->getError();

my $result = $cmdr->startRelease("Pet Store", "v2.1.0", {
    actualParameter => [{actualParameterName => "buildNumber", value => "1042"}],
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $frid = $result->findvalue('//flowRuntime/flowRuntimeId')->string_value();
```

---

## Stage and Task Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createStage` | Creates a stage in a pipeline | `projectName`, `stageName` |
| `createTask` | Creates a task in a stage | `projectName`, `taskName` |
| `completeManualTask` | Completes a waiting MANUAL task | `flowRuntimeId`, `stageName`, `taskName` |
| `retryTask` | Retries a failed task | `flowRuntimeId`, `stageName`, `taskName` |
| `getAllWaitingTasks` | Lists all tasks awaiting approval | `projectName` |
| `createGate` | Creates a PRE or POST gate on a stage | `projectName`, `stageName`, `gateType` |

---

### `completeManualTask`

```perl
$cmdr->completeManualTask($frid, "Deploy to Production", "Production Approval", {
    action => "completed",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```