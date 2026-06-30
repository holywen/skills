# CloudBees CD/RO Perl API — Job & Property Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Job Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `getJob` | Retrieves job metadata by job ID | `jobId` |
| `getJobs` | Retrieves all jobs with optional filtering | (none) |
| `abortJob` | Aborts a running job | `jobId` |
| `deleteJob` | Deletes a job | `jobId` |
| `waitForJob` | Blocks until job completes (Perl only) | `jobId` |
| `createJobStep` | Dynamically creates a step inside a running job | (none — all named) |
| `setJobName` | Renames a running job | `jobId`, `newName` |
| `abortAllJobs` | Aborts all running jobs server-wide | (none) |
| `abortJobStep` | Aborts a single running job step | `jobStepId` |

---

### `waitForJob`

Blocks until the job finishes. Perl API only.

```perl
my $result = $cmdr->runProcedure("Pet Store", {procedureName => "Deploy to QA"});
die $cmdr->getError()->{message} if $cmdr->getError();
my $jobId = $result->findvalue('//jobId')->string_value();

$cmdr->waitForJob($jobId, 600);  # 10 minute timeout
die $cmdr->getError()->{message} if $cmdr->getError();

my $outcome = $cmdr->getJob($jobId)->findvalue('//job/outcome')->string_value();
print "Outcome: $outcome\n";
```

---

### `createJobStep`

Dynamically adds a step to an already-running job.

```perl
$cmdr->createJobStep({
    jobStepId    => $ENV{COMMANDER_JOBSTEPID},
    projectName  => "Default",
    subprocedure => "Deploy",
    actualParameter => [
        {actualParameterName => "env",     value => "production"},
        {actualParameterName => "version", value => "2.5.1"},
    ],
    errorHandling => "abortJob",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## Property Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `setProperty` | Sets a property value | `propertyName`, `value` |
| `getProperty` | Retrieves a property value | `propertyName` |
| `getProperties` | Returns all properties on an object | locator |
| `deleteProperty` | Deletes a property | `propertyName` |
| `expandString` | Expands `$[...]` references in a string | `value` |
| `incrementProperty` | Atomically increments an integer property | `propertyName`, `incrementBy` |

---

### `setProperty` / `getProperty`

```perl
$cmdr->setProperty("BuildNumber", {
    value       => "1042",
    projectName => "Pet Store",
});
die $cmdr->getError()->{message} if $cmdr->getError();

my $build = $cmdr->getProperty("BuildNumber", {projectName => "Pet Store"})
                 ->findvalue('//value')->string_value();
print "Build: $build\n";
```

---

### `incrementProperty`

```perl
$cmdr->incrementProperty("GlobalBuildCounter", 1, {projectName => "Pet Store"});
die $cmdr->getError()->{message} if $cmdr->getError();
```