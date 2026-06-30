# CloudBees CD/RO Perl API — Resource & Environment Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Resource Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createResource` | Creates an agent resource | `resourceName` |
| `deleteResource` | Deletes a resource | `resourceName` |
| `getResource` | Retrieves a resource | `resourceName` |
| `modifyResource` | Modifies resource properties | `resourceName` |
| `createResourcePool` | Creates a resource pool | `resourcePoolName` |
| `addResourcesToPool` | Adds resources to a pool | `resourcePoolName` |
| `removeResourcesFromPool` | Removes resources from a pool | `resourcePoolName` |
| `getResourcesInPool` | Lists resources in a pool | `resourcePoolName` |
| `pingResource` | Pings a resource | `resourceName` |
| `pingAllResources` | Pings all resources | (none) |
| `getResourceUsage` | Returns resource usage stats | (none) |

---

### `createResource` + Pool management

```perl
$cmdr->createResource("build-agent-01", {
    hostName    => "build-host-01.example.com",
    description => "Primary Linux build agent",
    stepLimit   => 4,
    shell       => "bash",
    pools       => "linux-build-pool",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createResourcePool("linux-build-pool", {
    description  => "All Linux CI build agents",
    resourceName => ["build-agent-01", "build-agent-02"],
});
die $cmdr->getError()->{message} if $cmdr->getError();

# Ping to verify
my $result = $cmdr->pingResource("build-agent-01");
my $status = $result->findvalue('//resource/agentState/status')->string_value();
print "Agent status: $status\n";
```

---

## Environment Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createEnvironment` | Creates an environment | `projectName`, `environmentName` |
| `deleteEnvironment` | Deletes an environment | `projectName`, `environmentName` |
| `getEnvironment` | Retrieves an environment | `projectName`, `environmentName` |
| `modifyEnvironment` | Modifies an environment | `projectName`, `environmentName` |
| `createEnvironmentTier` | Creates a tier in an environment | `projectName`, `environmentName`, `environmentTierName` |
| `addEnvironmentTierResource` | Adds resource to a tier | `projectName`, `environmentName`, `environmentTierName`, `resourceName` |
| `addResourcesToEnvironmentTier` | Adds multiple resources to a tier | `projectName`, `environmentName`, `environmentTierName` |
| `removeResourcesFromEnvironmentTier` | Removes resources from a tier | `projectName`, `environmentName`, `environmentTierName` |
| `getResourcesInEnvironmentTier` | Lists resources in a tier | `projectName`, `environmentName`, `environmentTierName` |
| `getEnvironmentTiers` | Lists all tiers in an environment | `projectName`, `environmentName` |
| `provisionEnvironment` | Provisions a dynamic environment | named args |
| `tearDownResource` | Tears down a provisioned resource | `resourceName` |

---

### `createEnvironment` + `createEnvironmentTier`

```perl
$cmdr->createEnvironment("Pet Store", "Production US East", {
    description          => "Production cluster with rolling deploys",
    environmentEnabled   => "true",
    rollingDeployEnabled => "true",
    rollingDeployType    => "percentage",
    rollingDeployValue   => "25",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createEnvironmentTier("Pet Store", "Production US East", "App Tier", {
    description  => "Application servers",
    resourceName => ["app-server-01", "app-server-02"],
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->addResourcesToEnvironmentTier("Pet Store", "Production US East", "App Tier", {
    resourceName => ["app-server-03"],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```