# CloudBees CD/RO Perl API — Schedule & Artifact Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Schedule Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createSchedule` | Creates a time-based schedule | `projectName`, `scheduleName` |
| `deleteSchedule` | Deletes a schedule | `projectName`, `scheduleName` |
| `getSchedule` | Retrieves a schedule | `projectName`, `scheduleName` |
| `getSchedules` | Retrieves all schedules in a project | `projectName` |
| `modifySchedule` | Updates timing and targets | `projectName`, `scheduleName` |
| `runSchedule` | Immediately triggers a schedule | `projectName`, `scheduleName` |

---

### `createSchedule`

```perl
# Nightly build at 2 AM Mon-Fri
$cmdr->createSchedule("Pet Store", "Nightly Build", {
    description   => "Nightly full build",
    procedureName => "Build and Package WAR",
    startTime     => "02:00",
    stopTime      => "02:30",
    weekDays      => "Monday Tuesday Wednesday Thursday Friday",
    misfirePolicy => "runOnce",
    timeZone      => "America/New_York",
    actualParameter => [
        {actualParameterName => "buildProfile", value => "nightly"},
    ],
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Every 4 hours
$cmdr->createSchedule("Pet Store", "Integration Test Every 4h", {
    procedureName => "Run Integration Tests",
    interval      => 4,
    intervalUnits => "hours",
    startTime     => "00:00",
    stopTime      => "23:59",
    weekDays      => "Monday Tuesday Wednesday Thursday Friday Saturday Sunday",
    misfirePolicy => "skip",
    timeZone      => "UTC",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Pipeline schedule
$cmdr->createSchedule("Pet Store", "Weekly Release", {
    pipelineName => "Release Pipeline",
    startTime    => "08:00",
    stopTime     => "08:30",
    weekDays     => "Monday",
    timeZone     => "America/Chicago",
    priority     => "high",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `modifySchedule` / `runSchedule`

```perl
# Disable a schedule
$cmdr->modifySchedule("Pet Store", "Weekly Release", {
    scheduleDisabled => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Re-enable
$cmdr->modifySchedule("Pet Store", "Weekly Release", {
    scheduleDisabled => "false",
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Immediately trigger
$cmdr->runSchedule("Pet Store", "Nightly Build");
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Artifact Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createArtifact` | Creates an artifact record | `groupId`, `artifactKey` |
| `deleteArtifact` | Deletes an artifact and all versions | `artifactName` |
| `getArtifactVersions` | Retrieves artifact versions | (named) |
| `publishArtifactVersion` | Uploads files to a repository | (all named) |
| `retrieveArtifactVersions` | Downloads matching version | (all named) |
| `modifyArtifactVersion` | Changes version state | `artifactVersionName` |
| `updateArtifactVersion` | Adds files to an existing version | (all named) |
| `createRepository` | Creates an artifact repository | `repositoryName` |
| `getRepository` | Retrieves a repository | `repositoryName` |
| `modifyRepository` | Modifies repository settings | `repositoryName` |
| `createArtifactVersion` | Creates a version record | (named) |
| `addDependentsToArtifactVersion` | Adds dependency versions | (named) |
| `cleanupArtifactCache` | Removes cached artifacts from a resource | (named) |

---

### `publishArtifactVersion`

```perl
my $buildNum = $cmdr->getProperty("/myJob/buildNumber")
                    ->findvalue('//value')->string_value();
my $version  = "2.1.0-build.$buildNum";

my $result = $cmdr->publishArtifactVersion({
    artifactName    => "petstore:cart-service",
    version         => $version,
    repositoryName  => "internal-releases",
    fromDirectory   => "/workspace/target",
    includePatterns => "*.jar",
    excludePatterns => "*-sources.jar *-javadoc.jar",
    compressionType => "gz",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $artifactVersionName = $result->findvalue('//artifactVersion/artifactVersionName')->string_value();
print "Published: $artifactVersionName\n";

$cmdr->setProperty("/myJob/publishedArtifact", $artifactVersionName);
```

---

### `retrieveArtifactVersions`

```perl
my $result = $cmdr->retrieveArtifactVersions({
    artifactName => "petstore:cart-service",
    versionRange => "[2.0,3.0)",
    toDirectory  => "/opt/petstore/deploy",
    overwrite    => "forceOverwrite",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $retrieved = $result->findvalue('//artifactVersion/version')->string_value();
print "Retrieved: $retrieved\n";
```

---

### `modifyArtifactVersion`

```perl
# Mark a bad build as unavailable
$cmdr->modifyArtifactVersion("petstore:cart-service:2.1.0-build.108", {
    artifactVersionState => "unavailable",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```