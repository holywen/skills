# CloudBees CD/RO Groovy API — Credential, ACL & User

> Fetch per-command docs: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`

## Connection Setup

```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()
```

## Command Reference Table

| Command | Description | Required Args |
|---|---|---|
| `createCredential` | Creates a credential | `projectName`, `credentialName`, `userName`, `password` |
| `deleteCredential` | Deletes a credential | `projectName`, `credentialName` |
| `getCredential` | Returns credential metadata | `projectName`, `credentialName` |
| `modifyCredential` | Updates a credential | `projectName`, `credentialName` |
| `attachCredential` | Makes credential available to a step | `projectName`, `credentialName`, `procedureName` |
| `detachCredential` | Removes credential attachment | `projectName`, `credentialName`, `procedureName` |
| `createAclEntry` | Grants a privilege to a user/group | `principalType`, `principalName`, `objectType`, `objectId` |
| `deleteAclEntry` | Revokes a privilege | `principalType`, `principalName`, `objectType`, `objectId` |
| `getAcl` | Returns ACL entries for an object | `objectType`, `objectId` |
| `modifyAclEntry` | Updates privilege on an ACL entry | `principalType`, `principalName`, `objectType`, `objectId` |
| `createUser` | Creates a local user account | `userName` |
| `deleteUser` | Deletes a user | `userName` |
| `modifyUser` | Updates user attributes | `userName` |
| `createGroup` | Creates a user group | `groupName` |
| `addUserToGroup` | Adds a user to a group | `groupName`, `userName` |
| `removeUserFromGroup` | Removes a user from a group | `groupName`, `userName` |

---

### `createCredential` + `attachCredential`

```groovy
def result = ef.createCredential(
    projectName:    "PaymentService",
    credentialName: "nexus-deployer",
    userName:       System.getenv("NEXUS_USER"),
    password:       System.getenv("NEXUS_PASSWORD"),
    description:    "Nexus repository deployer account"
)
println "Created credential: ${result.credential.credentialName}"

// Attach to step (makes $[/myCredential/userName] available at runtime)
ef.attachCredential(
    projectName:    "PaymentService",
    credentialName: "nexus-deployer",
    procedureName:  "BuildAndTest",
    stepName:       "PublishArtifact"
)
```

---

### `createAclEntry` + `getAcl`

```groovy
// Grant dev-team read+execute on project
ef.createAclEntry(
    principalType:    "group",
    principalName:    "dev-team",
    objectType:       "project",
    objectId:         "PaymentService",
    readPrivilege:    "allow",
    executePrivilege: "allow",
    modifyPrivilege:  "deny"
)

// Inspect the ACL
def aclResult = ef.getAcl(
    objectType: "project",
    objectId:   "PaymentService"
)
aclResult.acl.aclEntry.each { entry ->
    println "${entry.principalType}:${entry.principalName}  read=${entry.readPrivilege}  execute=${entry.executePrivilege}"
}
```

---

### `createUser` + `createGroup` + `addUserToGroup`

```groovy
ef.createUser(
    userName:     "ci-automation",
    password:     System.getenv("CI_AUTOMATION_PASSWORD"),
    fullUserName: "CI Automation Service Account",
    email:        "ci-alerts@example.com"
)

ef.createGroup(
    groupName:   "release-managers",
    description: "Personnel authorized to approve production releases",
    groupType:   "local"
)

["alice.smith", "bob.jones"].each { user ->
    ef.addUserToGroup(groupName: "release-managers", userName: user)
    println "Added ${user} to release-managers"
}
```