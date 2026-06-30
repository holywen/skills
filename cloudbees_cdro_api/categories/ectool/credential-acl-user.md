# ectool — Credential, ACL & User

Fetch docs: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`

---

## Login / Session Setup

```bash
ectool login admin changeme --server cdro.example.com

export ELECTRIC_COMMANDER_SERVER=cdro.example.com
export COMMANDER_SESSIONID=<token>
```

---

## Command Reference Table

| Command | Description | Positional Args (in order) |
|---------|-------------|---------------------------|
| `createCredential` | Create a credential (username/password pair) | `projectName` `credentialName` |
| `modifyCredential` | Modify a credential | `projectName` `credentialName` |
| `deleteCredential` | Delete a credential | `projectName` `credentialName` |
| `getCredential` | Get credential metadata (not the secret) | `projectName` `credentialName` |
| `getCredentials` | List credentials in a project | `projectName` |
| `getFullCredential` | Get full credential including the password | _(none)_ |
| `attachCredential` | Attach credential to a procedure/step | `projectName` `credentialName` |
| `detachCredential` | Detach credential | `projectName` `credentialName` |
| `createAclEntry` | Create an access control entry | _(varies)_ |
| `modifyAclEntry` | Modify an ACL entry | _(varies)_ |
| `deleteAclEntry` | Delete an ACL entry | _(varies)_ |
| `getAcl` | Get ACL for an object | _(varies)_ |
| `breakAclInheritance` | Break ACL inheritance for an object | _(varies)_ |
| `restoreAclInheritance` | Restore ACL inheritance for an object | _(varies)_ |
| `checkAccess` | Check if the current user has a specific permission | _(none)_ |
| `getAccess` | Get access summary for an object | _(varies)_ |
| `changeOwner` | Change the owner of an object | _(varies)_ |
| `createUser` | Create a local user account | `userName` |
| `modifyUser` | Modify a user account | `userName` |
| `deleteUser` | Delete a user account | `userName` |
| `getUser` | Get user details | `userName` |
| `getUsers` | List all users | _(none)_ |
| `createGroup` | Create a local group | `groupName` |
| `modifyGroup` | Modify a group | `groupName` |
| `deleteGroup` | Delete a group | `groupName` |
| `getGroup` | Get group details | `groupName` |
| `getGroups` | List all groups | _(none)_ |
| `addUserToGroup` | Add a user to a group | `groupName` `userName` |
| `removeUserFromGroup` | Remove a user from a group | `groupName` `userName` |
| `addUsersToGroup` | Add multiple users to a group at once | _(none)_ |
| `removeUsersFromGroup` | Remove multiple users from a group | _(none)_ |
| `revokeUserAccessToken` | Revoke a specific user access token | _(none)_ |
| `revokeUserAccessTokens` | Revoke all access tokens for a user | _(none)_ |
| `getAccessTokens` | List access tokens for a user | _(none)_ |
| `deleteAccessToken` | Delete an access token | _(none)_ |

---

### `createCredential`

Creates a credential (username + secret password) in a project. Secrets are encrypted at rest.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project that owns the credential |
| `credentialName` | Yes | Unique credential name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--userName` | string | Username stored in the credential |
| `--password` | string | Password/secret (encrypted at rest) |
| `--description` | string | Human-readable description |

**Examples:**

```bash
# Create a Git credential
ectool createCredential MyProject git-credentials \
  --userName deploy-bot \
  --password "$(cat ~/.secrets/git-token)" \
  --description "GitHub deploy bot token"

# Create a database credential
ectool createCredential MyProject db-prod-credentials \
  --userName myapp_user \
  --password "S3cr3tP@ss!" \
  --description "Production database credentials"

# Create a Docker registry credential
ectool createCredential MyProject docker-registry \
  --userName ci-bot \
  --password "$(cat ~/.secrets/docker-token)" \
  --description "Docker Hub CI bot credentials"

# List credentials (metadata only, passwords are never returned)
ectool getCredentials MyProject --format json | python3 -c "
import sys, json
creds = json.load(sys.stdin).get('credential', [])
for c in creds:
    print(c['credentialName'], '-', c.get('userName', 'N/A'))
"
```

---

### `getFullCredential`

Returns the full credential record including the decrypted password value. Requires the calling user to have the `execute` privilege on the credential object. Use sparingly and only in automated contexts where the secret is immediately consumed.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--projectName` | string | Project owning the credential |
| `--credentialName` | string | Name of the credential to retrieve |

**Examples:**

```bash
# Retrieve the full credential (including password)
ectool getFullCredential \
  --projectName MyProject \
  --credentialName db-prod-credentials

# Extract just the password for use in a script
DB_PASS=$(ectool getFullCredential \
  --projectName MyProject \
  --credentialName db-prod-credentials \
  --format json | python3 -c "
import sys, json
cred = json.load(sys.stdin)['credential']
print(cred.get('password', ''))
")
echo "Connecting to DB..."
PGPASSWORD="$DB_PASS" psql -h db.prod.example.com -U myapp_user -d myapp
```

---

### `attachCredential`

Attaches a credential to a procedure or step so it can be accessed at runtime via `$[/myJob/credential/credName/userName]` and `$[/myJob/credential/credName/password]`.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `projectName` | Yes | Project owning the credential |
| `credentialName` | Yes | Credential to attach |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--procedureName` | string | Procedure to attach credential to |
| `--stepName` | string | Specific step (within a procedure) |
| `--credentialName` | string | Name used to reference at runtime |

**Examples:**

```bash
# Attach to an entire procedure
ectool attachCredential MyProject git-credentials \
  --procedureName Build

# Attach to a specific step
ectool attachCredential MyProject db-prod-credentials \
  --procedureName Deploy \
  --stepName "Run DB Migration"
```

---

### `createAclEntry`

Creates an access control entry on a CD/RO object, granting or denying privileges to a user or group.

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--principalType` | string | `user` or `group` |
| `--principalName` | string | User or group name |
| `--projectName` | string | Apply to a project |
| `--procedureName` | string | Apply to a procedure |
| `--pipelineName` | string | Apply to a pipeline |
| `--environmentName` | string | Apply to an environment |
| `--credentialName` | string | Apply to a credential |
| `--resourceName` | string | Apply to a resource |
| `--executePrivilege` | string | `allow`, `deny`, or `inherit` |
| `--readPrivilege` | string | `allow`, `deny`, or `inherit` |
| `--modifyPrivilege` | string | `allow`, `deny`, or `inherit` |
| `--changePermissionsPrivilege` | string | `allow`, `deny`, or `inherit` |

**Examples:**

```bash
# Grant a group read+execute on a project
ectool createAclEntry \
  --principalType group \
  --principalName dev-team \
  --projectName MyProject \
  --readPrivilege allow \
  --executePrivilege allow \
  --modifyPrivilege inherit

# Grant a user full access to a procedure
ectool createAclEntry \
  --principalType user \
  --principalName jane.doe \
  --projectName MyProject \
  --procedureName Deploy \
  --readPrivilege allow \
  --executePrivilege allow \
  --modifyPrivilege allow
```

---

### `createUser`

Creates a local user account.

**Positional Args:**

| Name | Required | Description |
|------|----------|-------------|
| `userName` | Yes | Login name |

**Named Options:**

| Flag | Type | Description |
|------|------|-------------|
| `--password` | string | Initial password |
| `--fullUserName` | string | Display name |
| `--email` | string | Email address |
| `--admin` | boolean | Grant server admin (1/0) |

**Examples:**

```bash
# Create a developer user
ectool createUser jane.doe \
  --fullUserName "Jane Doe" \
  --email jane.doe@example.com \
  --password "TempPass123!" \
  --admin 0

# Create a service account
ectool createUser ci-service-account \
  --fullUserName "CI Service Account" \
  --email ci@example.com \
  --password "$(openssl rand -base64 32)"
```

---

### `createGroup` and `addUserToGroup`

Create groups and assign users for team-based access control.

**Examples:**

```bash
# Create groups
ectool createGroup dev-team --description "Development team"
ectool createGroup ops-team --description "Operations team"

# Add users to groups
ectool addUserToGroup dev-team jane.doe
ectool addUserToGroup dev-team john.smith

# Remove from group
ectool removeUserFromGroup dev-team john.smith
```

---

### `revokeUserAccessTokens`

Revokes all access tokens for a user in a single call.

**Examples:**

```bash
# Revoke all tokens for a user
ectool revokeUserAccessTokens --userName jane.doe

# Offboard a departing employee
ectool revokeUserAccessTokens --userName former.employee
ectool modifyUser former.employee --userDisabled 1
```

---

## Shell Scripting Patterns

### Onboard a new project with RBAC

```bash
#!/usr/bin/env bash
set -euo pipefail

PROJECT="NewProject"
DEV_GROUP="dev-team"
OPS_GROUP="ops-team"

ectool createProject "$PROJECT" --description "New project with RBAC"

ectool createAclEntry \
  --principalType group --principalName "$DEV_GROUP" \
  --projectName "$PROJECT" \
  --readPrivilege allow --executePrivilege allow \
  --modifyPrivilege allow

ectool createAclEntry \
  --principalType group --principalName "$OPS_GROUP" \
  --projectName "$PROJECT" \
  --readPrivilege allow --executePrivilege allow \
  --modifyPrivilege allow --changePermissionsPrivilege allow

echo "RBAC configured for $PROJECT"
```

### Rotate a credential

```bash
#!/usr/bin/env bash
PROJECT="MyProject"
CRED="db-prod-credentials"
NEW_PASS="$(openssl rand -base64 32)"

ectool modifyCredential "$PROJECT" "$CRED" --password "$NEW_PASS"
echo "Credential '$CRED' rotated at $(date -u +%Y-%m-%dT%H:%M:%SZ)"
```
