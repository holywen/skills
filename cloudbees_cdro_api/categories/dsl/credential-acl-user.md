# DSL Reference: Credential, ACL Entry, User, Group

<!-- Fetch live docs:
  https://docs.cloudbees.com/apidocs/dsl/latest/c/credential
  https://docs.cloudbees.com/apidocs/dsl/latest/c/aclEntry
  https://docs.cloudbees.com/apidocs/dsl/latest/c/user
  https://docs.cloudbees.com/apidocs/dsl/latest/c/group
-->

## Running DSL

```bash
ectool evalDsl --dslFile credentials.groovy
ectool generateDsl --objectType credential --objectName "nexus-creds" --projectName "WebApp"
```

---

## Object Reference Table

| Object | Description | Required Properties |
|---|---|---|
| `credential` | Stored username/password or token used by steps and plugins | `name` |
| `aclEntry` | Permission entry granting a principal access to an object | `principalType`, `principalName` |
| `user` | CD/RO local user account | `userName` |
| `group` | Named collection of users for access control | `groupName` |

---

## credential

### Closure Style — project-scoped credential

```groovy
project "WebApp", {
    credential "nexus-creds", {
        description  "Nexus artifact repository service account"
        userName     "svc-nexus-deploy"
        password     "PLACEHOLDER"
    }

    credential "github-token", {
        description "GitHub personal access token"
        userName    "ci-bot"
        password    "PLACEHOLDER"
    }

    credential "aws-deploy", {
        description "AWS IAM access key for deployment"
        userName    "AKIAIOSFODNN7EXAMPLE"
        password    "PLACEHOLDER"
    }
}
```

### API Call Style

```groovy
createCredential(
    projectName:    "WebApp",
    credentialName: "nexus-creds",
    userName:       "svc-nexus-deploy",
    password:       "PLACEHOLDER",
    description:    "Nexus artifact repository service account"
)

modifyCredential(
    projectName:    "WebApp",
    credentialName: "nexus-creds",
    password:       "new-rotated-password"
)

def cred = getCredential(
    projectName:    "WebApp",
    credentialName: "nexus-creds"
)
println "User: ${cred.credential.userName}"
```

### Attaching credentials to procedures and steps

```groovy
project "WebApp", {
    credential "nexus-creds", {
        userName "svc-nexus"
        password "PLACEHOLDER"
    }

    procedure "Publish Artifact", {
        attachCredential(
            credentialName:     "nexus-creds",
            credentialVariable: "NEXUS"
        )

        step "Upload to Nexus", {
            command '''
                mvn deploy \\
                  -Drepository.username=$NEXUS_USER \\
                  -Drepository.password=$NEXUS_PASSWORD
            '''
            shell "/bin/bash"
        }
    }
}
```

### YAML DSL

```yaml
apiVersion: cloudbees.com/v2023.2
kind: project
metadata:
  name: WebApp
spec:
  credentials:
    - name: nexus-creds
      description: Nexus artifact repository service account
      userName: svc-nexus-deploy
```

> Passwords are never stored in YAML DSL files. Set them via `ectool modifyCredential` or a secrets manager integration after import.

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `userName` | String | Yes | Username portion of the credential |
| `password` | String | No | Password/token (write-only; never returned by get) |
| `description` | String | No | Human-readable description |

---

## aclEntry

### Closure Style — on a project

```groovy
project "WebApp", {
    aclEntry "user", "alice", {
        readPrivilege    "allow"
        modifyPrivilege  "allow"
        executePrivilege "allow"
        adminPrivilege   "deny"
    }

    aclEntry "group", "dev-team", {
        readPrivilege    "allow"
        modifyPrivilege  "allow"
        executePrivilege "allow"
        adminPrivilege   "deny"
    }

    aclEntry "group", "ops-team", {
        readPrivilege    "allow"
        modifyPrivilege  "deny"
        executePrivilege "allow"
        adminPrivilege   "deny"
    }
}
```

### Closure Style — on a credential (restrict access)

```groovy
project "WebApp", {
    credential "aws-prod", {
        userName "AKIAIOSFODNN7EXAMPLE"
        password "PLACEHOLDER"

        aclEntry "group", "ops-team", {
            readPrivilege    "allow"
            executePrivilege "allow"
            modifyPrivilege  "deny"
        }

        aclEntry "group", "dev-team", {
            readPrivilege    "deny"
            executePrivilege "deny"
            modifyPrivilege  "deny"
        }
    }
}
```

### API Call Style

```groovy
createAclEntry(
    principalType:    "group",
    principalName:    "release-managers",
    projectName:      "WebApp",
    procedureName:    "Deploy-Production",
    executePrivilege: "allow",
    readPrivilege:    "allow",
    modifyPrivilege:  "deny"
)

modifyAclEntry(
    principalType:    "group",
    principalName:    "dev-team",
    projectName:      "WebApp",
    readPrivilege:    "allow",
    executePrivilege: "allow"
)

deleteAclEntry(
    principalType: "group",
    principalName: "old-team",
    projectName:   "WebApp"
)
```

### Privilege Values

| Value | Meaning |
|---|---|
| `allow` | Explicitly grant |
| `deny` | Explicitly deny (overrides inherited allow) |
| `inherit` | Use the inherited value from parent (default) |

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `principalType` | String | Yes | `user` or `group` |
| `principalName` | String | Yes | User login name or group name |
| `readPrivilege` | String | No | `allow`, `deny`, `inherit` |
| `modifyPrivilege` | String | No | `allow`, `deny`, `inherit` |
| `executePrivilege` | String | No | `allow`, `deny`, `inherit` |
| `adminPrivilege` | String | No | `allow`, `deny`, `inherit` |

---

## user

### Closure Style

```groovy
user "alice", {
    fullUserName "Alice Johnson"
    email        "alice@example.com"
    password     "PLACEHOLDER"
    groups       ["dev-team", "project-leads"]
}
```

### API Call Style

```groovy
createUser(
    userName:     "alice",
    fullUserName: "Alice Johnson",
    email:        "alice@example.com",
    password:     "PLACEHOLDER"
)

modifyUser(
    userName: "alice",
    email:    "alice.johnson@example.com"
)

def u = getUser(userName: "alice")
println u.user.fullUserName
```

---

## group

### Closure Style

```groovy
group "dev-team", {
    description "Development team members"
    userNames   ["alice", "bob", "carol"]
}

group "release-managers", {
    description "Release management team"
    userNames   ["dave", "eve"]
}
```

### API Call Style

```groovy
createGroup(
    groupName:   "dev-team",
    description: "Development team members"
)

addUserToGroup(groupName: "dev-team", userName: "alice")
addUserToGroup(groupName: "dev-team", userName: "bob")

removeUserFromGroup(groupName: "dev-team", userName: "alice")

def grp = getGroup(groupName: "dev-team")
grp.group.userName.each { u -> println u }
```

### Key Properties

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | String | No | Group description |
| `userNames` | List | No | Initial list of member usernames |

---

## Full Example — project with credentials, groups, and ACLs

```groovy
group "dev-team", {
    description "All developers"
    userNames   ["alice", "bob", "carol"]
}

group "ops-team", {
    description "Operations engineers"
    userNames   ["dave", "eve"]
}

group "release-managers", {
    description "Release approval authority"
    userNames   ["frank"]
}

project "WebApp", {
    aclEntry "group", "dev-team", {
        readPrivilege    "allow"
        executePrivilege "allow"
        modifyPrivilege  "deny"
    }

    aclEntry "group", "ops-team", {
        readPrivilege    "allow"
        executePrivilege "allow"
        modifyPrivilege  "allow"
        adminPrivilege   "allow"
    }

    credential "nexus-creds", {
        description "Nexus service account"
        userName    "svc-nexus"
        password    "PLACEHOLDER"

        aclEntry "group", "dev-team", {
            readPrivilege    "allow"
            executePrivilege "allow"
        }
    }

    credential "aws-prod-deploy", {
        description "AWS production IAM key"
        userName    "AKIAIOSFODNN7EXAMPLE"
        password    "PLACEHOLDER"

        aclEntry "group", "dev-team", {
            readPrivilege    "deny"
            executePrivilege "deny"
        }
        aclEntry "group", "ops-team", {
            readPrivilege    "allow"
            executePrivilege "allow"
        }
    }

    procedure "Deploy-Production", {
        aclEntry "group", "release-managers", {
            executePrivilege "allow"
            readPrivilege    "allow"
        }
        aclEntry "group", "dev-team", {
            executePrivilege "deny"
            readPrivilege    "allow"
        }

        attachCredential(
            credentialName:     "aws-prod-deploy",
            credentialVariable: "AWS"
        )

        step "Deploy", {
            command '''
                AWS_ACCESS_KEY_ID=$AWS_USER \\
                AWS_SECRET_ACCESS_KEY=$AWS_PASSWORD \\
                aws ecs update-service --cluster prod --service webapp --force-new-deployment
            '''
            shell "/bin/bash"
        }
    }
}
```