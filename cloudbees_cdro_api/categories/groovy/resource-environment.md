# CloudBees CD/RO Groovy API — Resource & Environment

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createResource` | Creates an agent resource | `resourceName` |
| `deleteResource` | Deletes a resource | `resourceName` |
| `getResource` | Returns resource details | `resourceName` |
| `modifyResource` | Updates resource attributes | `resourceName` |
| `createResourcePool` | Creates a pool of resources | `resourcePoolName` |
| `modifyResourcePool` | Updates pool attributes | `resourcePoolName` |
| `pingResource` | Tests connectivity to a resource | `resourceName` |
| `moveResourceToPool` | Moves a resource into a pool | `resourceName`, `resourcePoolName` |
| `createEnvironment` | Creates a deployment environment | `projectName`, `environmentName` |
| `deleteEnvironment` | Deletes an environment | `projectName`, `environmentName` |
| `modifyEnvironment` | Updates environment attributes | `projectName`, `environmentName` |
| `createEnvironmentTier` | Creates a tier in an environment | `projectName`, `environmentName`, `environmentTierName` |
| `addEnvironmentTierResource` | Adds a resource to a tier | `projectName`, `environmentName`, `environmentTierName`, `resourceName` |
| `removeEnvironmentTierResource` | Removes a resource from a tier | `projectName`, `environmentName`, `environmentTierName`, `resourceName` |

---

### `createResource` + resource pool

```groovy
// Create the pool
ef.createResourcePool(
    resourcePoolName: "linux-build-pool",
    description:      "Linux CI build agents",
    orderingFilter:   "LEAST_CONNECTED"
)

// Create and pool agents
["01", "02", "03"].each { n ->
    ef.createResource(
        resourceName:     "linux-build-agent-${n}",
        hostName:         "build-agent-${n}.internal.example.com",
        resourcePoolName: "linux-build-pool",
        stepLimit:        4,
        shell:            "bash",
        zoneName:         "us-east-1"
    )
    println "Created: linux-build-agent-${n}"
}

// Ping all to verify
def resources = ef.getResources()
resources.resource.each { res ->
    def ping = ef.pingResource(resourceName: res.resourceName)
    println "${res.resourceName}: ${ping.resource?.agentState}"
}
```

---

### `createEnvironment` + tiers

```groovy
ef.createEnvironment(
    projectName:     "PaymentService",
    environmentName: "staging",
    description:     "Pre-production staging environment"
)

[[name: "app-tier", desc: "App servers"],
 [name: "db-tier",  desc: "Database servers"],
 [name: "lb-tier",  desc: "Load balancers"]].each { tier ->
    ef.createEnvironmentTier(
        projectName:         "PaymentService",
        environmentName:     "staging",
        environmentTierName: tier.name,
        description:         tier.desc
    )
}

["staging-app-01", "staging-app-02"].each { r ->
    ef.addEnvironmentTierResource(
        projectName:         "PaymentService",
        environmentName:     "staging",
        environmentTierName: "app-tier",
        resourceName:        r
    )
}
ef.addEnvironmentTierResource(
    projectName:         "PaymentService",
    environmentName:     "staging",
    environmentTierName: "db-tier",
    resourceName:        "staging-db-primary"
)
println "Staging environment configured"
```