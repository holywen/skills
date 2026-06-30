# REST API: Credentials, ACLs, Users, and Groups

## Overview

Credentials store usernames and passwords securely and can be attached to procedures, steps, or applications. ACLs (Access Control Lists) control which users and groups can read, modify, or execute objects. Users and groups provide authentication and role-based access. This file covers creating credentials, managing ACL entries, and working with users and groups.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List credentials | GET | `/projects/{projectName}/credentials` |
| Get a credential | GET | `/projects/{projectName}/credentials/{credentialName}` |
| Create a credential | POST | `/projects/{projectName}/credentials` |
| Update a credential | PUT | `/projects/{projectName}/credentials/{credentialName}` |
| Delete a credential | DELETE | `/projects/{projectName}/credentials/{credentialName}` |
| Attach credential | PUT | `/projects/{projectName}/procedures/{procedureName}/steps/{stepName}` |
| Get ACL | GET | `/acls` |
| Update ACL (create entry) | PUT | `/acls` |
| Delete ACL entry | DELETE | `/acls` |
| List users | GET | `/users` |
| Get a user | GET | `/users/{userName}` |
| Create a user | POST | `/users` |
| Update a user | PUT | `/users/{userName}` |
| Delete a user | DELETE | `/users/{userName}` |
| List groups | GET | `/groups` |
| Get a group | GET | `/groups/{groupName}` |
| Create a group | POST | `/groups` |
| Update a group | PUT | `/groups/{groupName}` |
| Delete a group | DELETE | `/groups/{groupName}` |
| Add user to group | PUT | `/groups/{groupName}` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `My Credential` → `My%20Credential`.

---

## 1. Create a Credential

**POST** `/projects/{projectName}/credentials`

Credentials are stored encrypted. The password is write-only — it cannot be retrieved via the API.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `credentialName` | string | Name for the credential |
| `userName` | string | Username (the credential subject) |
| `password` | string | Password (stored encrypted) |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `credentialType` | string | `local` (default) or `secret` |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/credentials" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "credentialName": "tomcat-deploy",
    "userName": "deployer",
    "password": "s3cr3tP@ss!",
    "description": "Tomcat deployment service account"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/credentials",
    credentialName="tomcat-deploy",
    userName="deployer",
    password="s3cr3tP@ss!",
    description="Tomcat deployment service account"
)
print(result["credential"]["credentialId"])
```

### Example JSON response

```json
{
  "credential": {
    "credentialId": "a1b2c3d4-aaaa-1111-2222-333300001234",
    "credentialName": "tomcat-deploy",
    "userName": "deployer",
    "projectName": "Online Banking",
    "description": "Tomcat deployment service account",
    "createTime": "2024-06-30T12:00:00.000Z"
  }
}
```

> **Security note:** The `password` field is never returned in GET responses. Treat credential IDs as sensitive — do not log or expose them unnecessarily.

---

## 2. Attach a Credential to a Step

**PUT** `/projects/{projectName}/procedures/{procedureName}/steps/{stepName}`

Credentials can be referenced by steps so the step has access to the username/password at runtime via `$[/myCredential/userName]` and `$[/myCredential/password]`.

### Required parameters (in request body)

| Parameter | Type | Description |
|---|---|---|
| `credentialReferenceParameters` | object | Map of parameter name to credential path |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/procedures/Deploy%20App/steps/Run%20Tomcat%20Deploy" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "credentialName": "tomcat-deploy",
    "attachedCredentials": [
      {"credentialName": "tomcat-deploy", "credentialReferenceParameter": "credential"}
    ]
  }'
```

### Python example

```python
import requests, urllib.parse

project = "Online Banking"
procedure = "Deploy App"
step = "Run Tomcat Deploy"
url = f"{cdro.base}/projects/{urllib.parse.quote(project)}/procedures/{urllib.parse.quote(procedure)}/steps/{urllib.parse.quote(step)}"

result = requests.put(url, headers=cdro.headers, json={
    "credentialName": "tomcat-deploy",
    "attachedCredentials": [
        {"credentialName": "tomcat-deploy", "credentialReferenceParameter": "credential"}
    ]
}, verify=False).json()
print("Credential attached")
```

---

## 3. Get an ACL

**GET** `/acls`

Returns ACL entries for a specified object. Objects are identified by type and name using query parameters.

### Required query parameters

| Parameter | Type | Description |
|---|---|---|
| `objectType` | string | Object type: `project`, `procedure`, `job`, `resource`, etc. |
| `objectId` | string | The object's UUID |

*Alternatively, use `projectName`, `procedureName`, etc. directly:*

| Parameter | Type | Description |
|---|---|---|
| `projectName` | string | Project-level ACL lookup |

### curl example

```bash
# Get ACL for a project
curl -k -X GET \
  "https://cdro.example.com/rest/v1.0/acls?projectName=Online%20Banking" \
  -H "sessionid: <API-token>"
```

### Python example

```python
result = cdro.get("acls", projectName="Online Banking")
for entry in result.get("aclEntry", []):
    print(entry.get("userName") or entry.get("groupName"),
          entry["readPrivilege"], entry["modifyPrivilege"])
```

### Example JSON response

```json
{
  "acl": {
    "aclId": "b2c3d4e5-bbbb-2222-3333-444400002345",
    "projectName": "Online Banking",
    "aclEntry": [
      {
        "principalType": "user",
        "userName": "admin",
        "readPrivilege": "allow",
        "modifyPrivilege": "allow",
        "executePrivilege": "allow"
      },
      {
        "principalType": "group",
        "groupName": "developers",
        "readPrivilege": "allow",
        "modifyPrivilege": "inherit",
        "executePrivilege": "allow"
      }
    ]
  }
}
```

---

## 4. Create a User

**POST** `/users`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `userName` | string | Login username |
| `password` | string | Initial password |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `fullUserName` | string | Display name |
| `email` | string | Email address |
| `ldapDomain` | string | LDAP/AD domain (for external auth) |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/users" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "userName": "jdoe",
    "password": "InitialP@ss1!",
    "fullUserName": "Jane Doe",
    "email": "jdoe@example.com"
  }'
```

### Python example

```python
result = cdro.post(
    "users",
    userName="jdoe",
    password="InitialP@ss1!",
    fullUserName="Jane Doe",
    email="jdoe@example.com"
)
print(result["user"]["userId"])
```

### Example JSON response

```json
{
  "user": {
    "userId": "c3d4e5f6-cccc-3333-4444-555500003456",
    "userName": "jdoe",
    "fullUserName": "Jane Doe",
    "email": "jdoe@example.com",
    "createTime": "2024-06-30T12:20:00.000Z"
  }
}
```

---

## 5. Create a Group and Add a User

**POST** `/groups`

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `groupName` | string | Name of the group |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Group description |
| `userNames` | array | Initial list of user names to include |

### curl example — create group

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/groups" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "groupName": "developers",
    "description": "Application development team"
  }'
```

### Add user to group (PUT action)

**PUT** `/groups/{groupName}`

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/groups/developers" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "addUsers": ["jdoe", "asmith"]
  }'
```

### Python example — full flow

```python
import requests

# Create group
cdro.post("groups", groupName="developers", description="Application development team")

# Add users
url = f"{cdro.base}/groups/developers"
result = requests.put(url, headers=cdro.headers,
                      json={"addUsers": ["jdoe", "asmith"]}, verify=False).json()
print(result["group"]["groupName"])
```

### Example JSON response (add users)

```json
{
  "group": {
    "groupId": "d4e5f6a7-dddd-4444-5555-666600004567",
    "groupName": "developers",
    "description": "Application development team",
    "user": [
      {"userName": "jdoe"},
      {"userName": "asmith"}
    ]
  }
}
```

---

## Notes

- **Password security:** Passwords in POST/PUT requests are transmitted over HTTPS and stored encrypted server-side. Never store API responses containing passwords, and never log request bodies that include passwords.
- **ACL privileges:** Valid privilege values are `allow`, `deny`, and `inherit`. The `inherit` value defers to the parent object's ACL.
- **Principal types:** ACL entries can apply to a `user` or `group` principal. Use `groupName` for group entries, `userName` for user entries.
- **LDAP users:** For LDAP-authenticated users, specify the `ldapDomain` parameter. The password field may be omitted for pure LDAP users.
- **URI encoding:** All path segments with spaces must be percent-encoded. Python: `urllib.parse.quote("My Group")`.
- **Swagger UI:** Full parameter reference including ACL privilege types and credential attachment details at `https://<cdro-server>/rest/doc/v1.0/`.