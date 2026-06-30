# DSL Reference: Schedule, Artifact, ArtifactVersion

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/schedule
  https://docs.cloudbees.com/apidocs/dsl/latest/c/artifact
  https://docs.cloudbees.com/apidocs/dsl/latest/c/artifactVersion
-->

## Running DSL

```bash
ectool evalDsl --dslFile schedule.groovy
ectool generateDsl --objectType schedule --objectName "Nightly-Build" --projectName "WebApp"
ectool generateDsl --objectType artifact  --objectName "com.example:backend"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `schedule` | Cron-based or event-driven trigger for a procedure or pipeline | `name` |
| `artifact` | Named artifact definition (group:name identifier) | `name` (groupId:artifactKey) |
| `artifactVersion` | Specific published version of an artifact | `version` |

---

## schedule

### Closure Style — cron expression schedule

```groovy
project "WebApp", {
    schedule "Weekday-Build", {
        description   "Build Mon-Fri at 6am UTC"
        procedureName "Build"
        cronExpression "0 6 * * 1-5"
        enabled       true
        priority      "normal"

        actualParameter [
            [actualParameterName: "branch", value: "main"]
        ]
    }
}
```

### Closure Style — pipeline schedule

```groovy
project "WebApp", {
    schedule "Nightly-Pipeline", {
        description    "Run full CI/CD pipeline nightly"
        pipelineName   "CI-CD"
        cronExpression "0 2 * * *"
        enabled        true

        actualParameter [
            [actualParameterName: "branch",    value: "main"],
            [actualParameterName: "targetEnv", value: "staging"]
        ]
    }
}
```

### Closure Style — interval schedule

```groovy
project "WebApp", {
    schedule "SCM-Poll", {
        description   "Poll Git repo every 5 minutes"
        procedureName "Build"
        interval      "5"
        intervalUnits "minutes"
        enabled       true
        misfirePolicy "ignore"

        actualParameter [
            [actualParameterName: "branch", value: "main"]
        ]
    }
}
```

### API Call Style

```groovy
createSchedule(
    projectName:   "WebApp",
    scheduleName:  "Nightly-Build",
    procedureName: "Build",
    startTime:     "02:00",
    interval:      "1",
    intervalUnits: "days",
    enabled:       true,
    description:   "Nightly CI build"
)

runSchedule(
    projectName:  "WebApp",
    scheduleName: "Nightly-Build"
)

modifySchedule(
    projectName:  "WebApp",
    scheduleName: "Nightly-Build",
    enabled:      false
)

def sched = getSchedule(
    projectName:  "WebApp",
    scheduleName: "Nightly-Build"
)
println "Next run: ${sched.schedule.nextRunTime}"
println "Enabled:  ${sched.schedule.enabled}"
```

### YAML DSL

```yaml
apiVersion: cloudbees.com/v2023.2
kind: project
metadata:
  name: WebApp
spec:
  schedules:
    - name: Nightly-Build
      description: Run full CI build nightly
      procedureName: Build
      cronExpression: "0 2 * * *"
      enabled: true
      actualParameters:
        - actualParameterName: branch
          value: main
        - actualParameterName: skipTests
          value: "false"
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `procedureName` | String | Cond. | Procedure to trigger |
| `pipelineName` | String | Cond. | Pipeline to trigger |
| `cronExpression` | String | Cond. | 5-field cron expression |
| `startTime` | String | Cond. | HH:MM start time (used with `interval`) |
| `interval` | String | Cond. | Numeric interval (used with `intervalUnits`) |
| `intervalUnits` | String | No | `minutes`, `hours`, `days`, `weeks` |
| `timeZone` | String | No | Java timezone ID, e.g. `America/New_York` |
| `enabled` | Boolean | No | Whether schedule fires |
| `priority` | String | No | `low`, `normal`, `high` |
| `misfirePolicy` | String | No | `ignore` or `runOnce` when a fire is missed |

---

## artifact

### Closure Style

```groovy
artifact "com.example:backend", {
    description    "Spring Boot backend service JAR"
    artifactKey    "backend"
    groupId        "com.example"
    repositoryName "artifact-repo"
}

artifact "com.example:frontend", {
    description    "React SPA static bundle"
    artifactKey    "frontend"
    groupId        "com.example"
    repositoryName "artifact-repo"
}
```

### API Call Style

```groovy
createArtifact(
    artifactName:   "com.example:backend",
    description:    "Spring Boot backend service JAR",
    repositoryName: "artifact-repo"
)

def art = getArtifact(artifactName: "com.example:backend")
println art.artifact.description

deleteArtifact(artifactName: "com.example:backend")
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `artifactKey` | String | Yes | Short name component of `groupId:artifactKey` |
| `groupId` | String | Yes | Group or org identifier |
| `repositoryName` | String | No | Target artifact repository |
| `description` | String | No | Human-readable description |

---

## artifactVersion

### API Call Style — publishArtifactVersion

```groovy
publishArtifactVersion(
    artifactName:    "com.example:backend",
    version:         "3.0.1",
    repositoryName:  "artifact-repo",
    fromDirectory:   "/workspace/webapp/target",
    includePatterns: "backend-*.jar"
)

publishArtifactVersion(
    artifactName:    "com.example:release-bundle",
    version:         "3.0.1",
    repositoryName:  "artifact-repo",
    fromDirectory:   "/workspace/dist",
    includePatterns: "*.jar,*.war,*.zip",
    excludePatterns: "*-sources.jar"
)
```

### API Call Style — retrieveArtifactVersions

```groovy
retrieveArtifactVersions(
    artifactVersionName: "com.example:backend:3.0.1",
    toDirectory:         "/opt/deploy/backend"
)

def result = retrieveArtifactVersions(
    artifactVersionName: "com.example:backend:LATEST",
    toDirectory:         "/opt/deploy/backend"
)
println "Retrieved version: ${result.artifactVersion.version}"
```

### API Call Style — getArtifactVersions / deleteArtifactVersion

```groovy
def versions = getArtifactVersions(artifactName: "com.example:backend")
versions.artifactVersion.each { v ->
    println "${v.version}  created: ${v.createTime}"
}

deleteArtifactVersion(
    artifactName:    "com.example:backend",
    artifactVersion: "2.0.0"
)

def cutoff = "2024-01-01T00:00:00Z"
versions.artifactVersion.findAll { it.createTime < cutoff }.each { v ->
    deleteArtifactVersion(
        artifactName:    "com.example:backend",
        artifactVersion: v.version
    )
}
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `version` | String | Yes | Version string, e.g. `3.0.1`, `LATEST`, `3.0.*` |
| `repositoryName` | String | No | Repository to publish to |
| `fromDirectory` | String | Cond. | Source directory for publish |
| `includePatterns` | String | No | Glob patterns for files to include |
| `excludePatterns` | String | No | Glob patterns for files to exclude |
| `toDirectory` | String | Cond. | Destination directory for retrieve |

---

## Full Example — build procedure with schedule and artifact publish

```groovy
artifact "com.example:backend", {
    description    "Spring Boot backend JAR"
    repositoryName "artifact-repo"
}

project "WebApp", {
    procedure "Build-and-Publish", {
        description "Compile, test, and publish artifact"

        formalParameter "branch", {
            defaultValue "main"
            required     true
            type         "entry"
        }

        formalOutputParameter "artifactVersion", {
            description "Published artifact version string"
        }

        step "Checkout", {
            command  "git clone -b \$[branch] https://git.example.com/webapp.git /workspace/webapp"
            shell    "/bin/bash"
        }

        step "Build", {
            command          "mvn clean package -DskipTests=false"
            shell            "/bin/bash"
            workingDirectory "/workspace/webapp"
        }

        step "Determine Version", {
            command '''
                VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout -f /workspace/webapp/pom.xml)
                BUILD="${VERSION}-b${BUILD_NUMBER:-$(date +%Y%m%d%H%M%S)}"
                echo "VERSION=$BUILD" > /workspace/version.env
                ectool setOutputParameter artifactVersion "$BUILD"
            '''
            shell "/bin/bash"
        }

        step "Publish Artifact", {
            command '''
                source /workspace/version.env
                ectool publishArtifactVersion \\
                  --artifactName com.example:backend \\
                  --version "$VERSION" \\
                  --repositoryName artifact-repo \\
                  --fromDirectory /workspace/webapp/target \\
                  --includePatterns "backend-*.jar"
            '''
            shell "/bin/bash"
        }
    }

    schedule "Nightly-Build-and-Publish", {
        description    "Nightly artifact build at 3am UTC"
        procedureName  "Build-and-Publish"
        cronExpression "0 3 * * *"
        enabled        true
        priority       "normal"

        actualParameter [
            [actualParameterName: "branch", value: "main"]
        ]
    }
}
```