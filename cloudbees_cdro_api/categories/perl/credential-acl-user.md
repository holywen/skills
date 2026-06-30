# CloudBees CD/RO Perl API — Credential, ACL & User Management

> Per-command docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

## Connection Setup

```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();
```

## Credential Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createCredential` | Creates a named credential | `projectName`, `credentialName`, `userName`, `password` |
| `deleteCredential` | Deletes a credential | `projectName`, `credentialName` |
| `getCredential` | Retrieves credential metadata | `projectName`, `credentialName` |
| `modifyCredential` | Modifies a credential | `projectName`, `credentialName` |
| `attachCredential` | Attaches credential to a step/schedule | `projectName`, `credentialName` |
| `detachCredential` | Detaches a credential | `projectName`, `credentialName` |
| `getFullCredential` | Gets plaintext credentials (inside running step) | `credentialName` |

---

### `createCredential` + `attachCredential`

```perl
$cmdr->createCredential("Pet Store", "nexus-deploy", "ci-deployer", "S3cur3P@ss!", {
    description => "Nexus repository credentials",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->attachCredential("Pet Store", "nexus-deploy", {
    procedureName => "Build and Package WAR",
    stepName      => "Publish to Nexus",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

### `getFullCredential` (inside running step)

```perl
my $xpath    = $cmdr->getFullCredential("nexus-deploy");
die $cmdr->getError()->{message} if $cmdr->getError();
my $username = $xpath->findvalue('//credential/userName')->string_value();
my $password = $xpath->findvalue('//credential/password')->string_value();
```

---

## ACL Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createAclEntry` | Creates an ACE on an object | `principalType`, `principalName` + locator |
| `getAcl` | Retrieves full ACL for an object | locator |
| `modifyAclEntry` | Modifies an ACE | `principalType`, `principalName` + locator |
| `breakAclInheritance` | Breaks parent ACL inheritance | locator |
| `restoreAclInheritance` | Restores ACL inheritance | locator |
| `checkAccess` | Returns effective permissions | locator |

---

### `createAclEntry`

```perl
$cmdr->createAclEntry("group", "release-managers", {
    projectName                => "Pet Store",
    pipelineName               => "Release Pipeline",
    readPrivilege              => "allow",
    modifyPrivilege            => "allow",
    executePrivilege           => "allow",
    changePermissionsPrivilege => "allow",
});
die $cmdr->getError()->{message} if $cmdr->getError();
```

---

## User & Group Command Reference

| Command | Description | Required Positional Args |
|---------|-------------|---------------------------|
| `createUser` | Creates a local user | `userName` |
| `deleteUser` | Deletes a user | `userName` |
| `modifyUser` | Modifies a user | `userName` |
| `createGroup` | Creates a local group | `groupName` |
| `deleteGroup` | Deletes a group | `groupName` |
| `addUserToGroup` | Adds a user to a group | `groupName`, `userName` |
| `removeUserFromGroup` | Removes a user from a group | `groupName`, `userName` |
| `addUsersToGroup` | Adds multiple users to a group | `groupName` |
| `removeUsersFromGroup` | Removes multiple users from a group | `groupName` |

---

### `createUser` + `createGroup`

```perl
$cmdr->createUser("alice.chen", {
    fullUserName => "Alice Chen",
    email        => "alice@example.com",
    password     => "InitialP@ss1!",
    persona      => "developer",
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->createGroup("release-managers", {
    description => "Team responsible for production releases",
    userNames   => ["alice.chen"],
});
die $cmdr->getError()->{message} if $cmdr->getError();

$cmdr->addUsersToGroup("release-managers", {
    userName => ["bob.jones", "charlie.smith"],
});
die $cmdr->getError()->{message} if $cmdr->getError();
```