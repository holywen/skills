# DSL Reference: Resource, ResourcePool, Environment, EnvironmentTier, ResourceTemplate

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/resource
  https://docs.cloudbees.com/apidocs/dsl/latest/c/resourcePool
  https://docs.cloudbees.com/apidocs/dsl/latest/c/environment
  https://docs.cloudbees.com/apidocs/dsl/latest/c/environmentTier
  https://docs.cloudbees.com/apidocs/dsl/latest/c/resourceTemplate
-->

## Running DSL

```bash
ectool evalDsl --dslFile resources.groovy
ectool generateDsl --objectType resource    --objectName "build-agent-01"
ectool generateDsl --objectType environment --objectName "staging" --projectName "WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `resource` | Agent host that executes procedure steps | `name` |
| `resourcePool` | Named set of resources used for load-balancing | `name` |
| `environment` | Named deployment target grouping tiers and resources | `name` |
| `environmentTier` | Logical tier within an environment (web, app, db) | `name` |
| `resourceTemplate` | Cloud/dynamic resource provisioning template | `name` |

---

## resource

### Closure Style

```groovy
resource "build-agent-01", {
    description      "Linux x64 build agent — rack A"
    hostName         "build-agent-01.internal.example.com"
    port             7800
    shell            "/bin/bash"
    workspaceName    "default"
    pools            "build-agents,linux-agents"
    enabled          true
    repositoryNames  "artifact-repo"

    property "os",   value: "rhel9"
    property "arch", value: "x86_64"
    property "java", value: "17"
}
```

### API Call Style

```groovy
createResource(
    resourceName: "build-agent-01",
    hostName:     "build-agent-01.internal.example.com",
    port:         7800,
    description:  "Linux x64 build agent",
    pools:        "build-agents"
)

def ping = pingResource(resourceName: "build-agent-01")
println "Alive: ${ping.resource.alive}"

modifyResource(resourceName: "build-agent-01", enabled: false)
deleteResource(resourceName: "build-agent-01")
```

### YAML DSL

```yaml
apiVersion: cloudbees.com/v2023.2
kind: resource
metadata:
  name: build-agent-01
spec:
  description: Linux x64 build agent
  hostName: build-agent-01.internal.example.com
  port: 7800
  shell: /bin/bash
  pools: build-agents
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `hostName` | String | Yes | DNS name or IP of the agent host |
| `port` | Integer | No | Agent port (default 7800) |
| `shell` | String | No | Default shell for steps on this resource |
| `pools` | String | No | Comma-separated resource pool names |
| `workspaceName` | String | No | Default workspace |
| `enabled` | Boolean | No | Whether resource accepts work |
| `repositoryNames` | String | No | Artifact repositories available to this resource |

---

## resourcePool

### Closure Style

```groovy
resourcePool "build-agents", {
    description  "Pool of Linux build agents"
    orderByUsage true

    resource "build-agent-01", {
        hostName "build-agent-01.internal.example.com"
        port     7800
    }
    resource "build-agent-02", {
        hostName "build-agent-02.internal.example.com"
        port     7800
    }
    resource "build-agent-03", {
        hostName "build-agent-03.internal.example.com"
        port     7800
    }
}
```

### API Call Style

```groovy
createResourcePool(
    resourcePoolName: "build-agents",
    description:      "Pool of Linux build agents",
    orderByUsage:     true
)

addResourcesToPool(
    resourcePoolName: "build-agents",
    resourceName:     ["build-agent-01", "build-agent-02"]
)

def pool = getResourcePool(resourcePoolName: "build-agents")
println pool.resourcePool.resourceName
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Pool description |
| `orderByUsage` | Boolean | No | Prefer least-recently-used resource |

---

## environment

### Closure Style

```groovy
project "WebApp", {
    environment "staging", {
        description "Staging environment for pre-production validation"

        property "baseUrl",   value: "https://staging.example.com"
        property "dbHost",    value: "db-staging.internal"
        property "dbPort",    value: "5432"
        property "dbName",    value: "webapp_staging"
        property "appServer", value: "app-staging-01"

        environmentTier "Web", {
            resource "nginx-staging-01", {
                hostName "nginx-staging-01.internal.example.com"
                port     7800
            }
        }

        environmentTier "App", {
            resource "app-staging-01", {
                hostName "app-staging-01.internal.example.com"
                port     7800
            }
            resource "app-staging-02", {
                hostName "app-staging-02.internal.example.com"
                port     7800
            }
        }

        environmentTier "Data", {
            resource "db-staging-01", {
                hostName "db-staging-01.internal.example.com"
                port     7800
            }
        }
    }

    environment "production", {
        description "Production environment"

        property "baseUrl",   value: "https://app.example.com"
        property "dbHost",    value: "db-prod.internal"
        property "appServer", value: "app-prod-01"

        environmentTier "Web", {
            resource "nginx-prod-01", { hostName "nginx-prod-01.internal"; port 7800 }
            resource "nginx-prod-02", { hostName "nginx-prod-02.internal"; port 7800 }
        }

        environmentTier "App", {
            resourcePool "prod-app-pool", {
                resource "app-prod-01", { hostName "app-prod-01.internal"; port 7800 }
                resource "app-prod-02", { hostName "app-prod-02.internal"; port 7800 }
                resource "app-prod-03", { hostName "app-prod-03.internal"; port 7800 }
            }
        }
    }
}
```

### API Call Style

```groovy
createEnvironment(
    projectName:     "WebApp",
    environmentName: "staging",
    description:     "Staging environment"
)

createEnvironmentTier(
    projectName:         "WebApp",
    environmentName:     "staging",
    environmentTierName: "App"
)

addEnvironmentTierResource(
    projectName:         "WebApp",
    environmentName:     "staging",
    environmentTierName: "App",
    resourceName:        "app-staging-01"
)

def env = getEnvironment(
    projectName:     "WebApp",
    environmentName: "staging"
)
println env.environment.description

def resources = getEnvironmentTierResources(
    projectName:         "WebApp",
    environmentName:     "staging",
    environmentTierName: "App"
)
resources.resource.each { r ->
    def ping = pingResource(resourceName: r.resourceName)
    println "${r.resourceName}: ${ping.resource.alive ? 'UP' : 'DOWN'}"
}
```

### YAML DSL

```yaml
apiVersion: cloudbees.com/v2023.2
kind: project
metadata:
  name: WebApp
spec:
  environments:
    - name: staging
      description: Staging environment
      environmentTiers:
        - name: App
          resources:
            - resourceName: app-staging-01
            - resourceName: app-staging-02
        - name: Web
          resources:
            - resourceName: nginx-staging-01
```

---

## environmentTier

### API Call Style

```groovy
addEnvironmentTierResource(
    projectName:         "WebApp",
    environmentName:     "staging",
    environmentTierName: "App",
    resourceName:        "app-staging-03"
)

removeEnvironmentTierResource(
    projectName:         "WebApp",
    environmentName:     "staging",
    environmentTierName: "App",
    resourceName:        "app-staging-03"
)
```

---

## resourceTemplate

Resource templates enable dynamic provisioning (cloud VMs, containers) on demand.

### Closure Style

```groovy
resourceTemplate "aws-linux-agent", {
    description    "On-demand AWS EC2 build agent"
    pluginKey      "EC-EC2"
    image          "ami-0abcdef1234567890"
    instanceType   "t3.large"
    region         "us-east-1"
    securityGroups "sg-0abc123def456"
    subnetId       "subnet-0abc123"
    keyPair        "cloudbees-agents"

    property "agentLabel", value: "aws-linux"
    property "javaHome",   value: "/usr/lib/jvm/java-17"
}
```

### API Call Style

```groovy
createResourceTemplate(
    resourceTemplateName: "aws-linux-agent",
    pluginKey:            "EC-EC2",
    description:          "On-demand AWS EC2 build agent"
)

def res = createResourceFromTemplate(
    resourceTemplateName: "aws-linux-agent",
    resourceName:         "dynamic-agent-01"
)
println "Provisioned: ${res.resource.resourceName}"
```

---

## Full Nesting Example

```groovy
project "WebApp", {
    resourcePool "build-agents", {
        description "Shared CI build pool"
        resource "build-01", { hostName "build-01.internal"; port 7800 }
        resource "build-02", { hostName "build-02.internal"; port 7800 }
    }

    environment "staging", {
        description "Staging: pre-production validation target"

        property "baseUrl",   value: "https://staging.example.com"
        property "dbHost",    value: "db-staging.internal"
        property "namespace", value: "webapp-staging"

        environmentTier "Web", {
            resource "nginx-stg-01", {
                hostName "nginx-stg-01.internal"
                port     7800
                property "role", value: "loadbalancer"
            }
        }

        environmentTier "App", {
            resource "app-stg-01", { hostName "app-stg-01.internal"; port 7800 }
            resource "app-stg-02", { hostName "app-stg-02.internal"; port 7800 }
        }

        environmentTier "Data", {
            resource "db-stg-01", {
                hostName "db-stg-01.internal"
                port     7800
                property "dbPort", value: "5432"
            }
        }
    }

    procedure "Ping All Staging Resources", {
        step "Check Connectivity", {
            command '''
                ectool pingResource nginx-stg-01
                ectool pingResource app-stg-01
                ectool pingResource app-stg-02
                ectool pingResource db-stg-01
            '''
            shell "/bin/bash"
        }
    }
}
```