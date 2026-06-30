# CloudBees CD/RO Perl API — Project, Procedure & Step

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`
> Example: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/createProject`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
# Uses ELECTRIC_COMMANDER_SERVER env var (default: localhost)
# Or specify explicitly:
my $cmdr = new ElectricCommander({server => 'cdro.example.com', port => 443});
```

## Reading Response Values

```perl
my $result = $cmdr->getProject("MyProject");
die $cmdr->getError()->{message} if $cmdr->getError();
my $projectId   = $result->findvalue('//project/projectId')->string_value();
my $description = $result->findvalue('//project/description')->string_value();
```

---

## Project Management Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createProject` | Creates a new project; name must be unique system-wide | `projectName` |
| `deleteProject` | Permanently deletes a project and all child objects | `projectName` |
| `getProject` | Retrieves a project by name | `projectName` |
| `getProjects` | Retrieves all projects in the system | (none) |
| `modifyProject` | Modifies project properties | `projectName` |
| `copyProject` | Copies a project to a new name | `projectName`, `newProjectName` |

---

### `createProject`

Creates a new top-level project.

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();

$cmdr->createProject("Pet Store", {
    description   => "Production Pet Store release pipeline",
    workspaceName => "Primary",
    resourceName  => "build-agent-01",
    tracked       => "true",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `runProcedure`

**Key Named Options**

| Option | Type | Description |
|--------|------|-------------|
| `procedureName` | string | Name of procedure to run |
| `actualParameter` | hash/array | Runtime parameters |
| `priority` | string | `low`, `normal`, `high` |

```perl
my $result = $cmdr->runProcedure("Pet Store", {
    procedureName   => "Deploy to QA",
    actualParameter => [
        {actualParameterName => "artifactVersion", value => "1.4.7-build.112"},
        {actualParameterName => "targetEnv",       value => "qa-east"},
    ],
    priority => "high",
});
die $cmdr->getError()->{message} if $cmdr->getError();
my $jobId = $result->findvalue('//jobId')->string_value();
print "Started job: $jobId\n";
```

---

### `createStep`

```perl
$cmdr->createStep("Pet Store", "Build and Package WAR", "Compile Sources", {
    command        => "mvn clean package -DskipTests",
    shell          => "bash",
    resourceName   => "linux-build-pool",
    errorHandling  => "failProcedure",
    timeLimit      => "20",
    timeLimitUnits => "minutes",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```