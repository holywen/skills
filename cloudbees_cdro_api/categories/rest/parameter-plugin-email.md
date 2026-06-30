# REST API: Formal Parameters, Plugins, and Email Configurations

## Overview

Formal parameters define the inputs a procedure, pipeline, or application process accepts. Plugins extend CloudBees CD/RO with additional integrations (e.g., EC-Git, EC-Docker). Email configurations enable notification delivery. This file covers creating parameters, installing and promoting plugins, and configuring email.

**Base URL:** `https://<cdro-server>/rest/v1.0/`
**Swagger UI (full reference):** `https://<cdro-server>/rest/doc/v1.0/`

---

## URL Pattern Reference

| Operation | Method | Endpoint |
|---|---|---|
| List formal parameters | GET | `/projects/{projectName}/procedures/{procedureName}/formalParameters` |
| Get a formal parameter | GET | `/projects/{projectName}/procedures/{procedureName}/formalParameters/{paramName}` |
| Create a formal parameter | POST | `/projects/{projectName}/procedures/{procedureName}/formalParameters` |
| Update a formal parameter | PUT | `/projects/{projectName}/procedures/{procedureName}/formalParameters/{paramName}` |
| Delete a formal parameter | DELETE | `/projects/{projectName}/procedures/{procedureName}/formalParameters/{paramName}` |
| List pipeline parameters | GET | `/projects/{projectName}/pipelines/{pipelineName}/formalParameters` |
| Create pipeline parameter | POST | `/projects/{projectName}/pipelines/{pipelineName}/formalParameters` |
| List plugins | GET | `/plugins` |
| Get a plugin | GET | `/plugins/{pluginName}` |
| Install a plugin | POST | `/plugins` |
| Promote a plugin | PUT | `/plugins/{pluginName}` |
| Delete a plugin | DELETE | `/plugins/{pluginName}` |
| List email configs | GET | `/emailConfigs` |
| Get an email config | GET | `/emailConfigs/{configName}` |
| Create an email config | POST | `/emailConfigs` |
| Update an email config | PUT | `/emailConfigs/{configName}` |
| Delete an email config | DELETE | `/emailConfigs/{configName}` |
| Send a test email | PUT | `/emailConfigs/{configName}` |

> **URI encoding:** Spaces in names must be percent-encoded. Example: `Deploy App` → `Deploy%20App`.

---

## 1. Create a Formal Parameter

**POST** `/projects/{projectName}/procedures/{procedureName}/formalParameters`

Formal parameters define the interface for a procedure. When the procedure is called, actual parameters supply the values.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `formalParameterName` | string | Name of the parameter |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Human-readable description |
| `defaultValue` | string | Default value if not provided by caller |
| `required` | boolean | Whether a value must be supplied (default: `false`) |
| `type` | string | `String` (default), `Integer`, `Float`, `Boolean`, `Credential`, `TextArea`, `Select`, `Checkbox` |
| `label` | string | Display label in UI |
| `expansionDeferred` | boolean | Defer property expansion to runtime |
| `optionsDSL` | string | DSL expression to populate select/checkbox options |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/projects/Online%20Banking/procedures/Deploy%20App/formalParameters" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "formalParameterName": "environment",
    "description": "Target deployment environment",
    "defaultValue": "staging",
    "required": true,
    "type": "Select",
    "label": "Environment"
  }'
```

### Python example

```python
import urllib.parse

project = "Online Banking"
procedure = "Deploy App"
result = cdro.post(
    f"projects/{urllib.parse.quote(project)}/procedures/{urllib.parse.quote(procedure)}/formalParameters",
    formalParameterName="environment",
    description="Target deployment environment",
    defaultValue="staging",
    required=True,
    type="Select",
    label="Environment"
)
print(result["formalParameter"]["formalParameterId"])
```

### Example JSON response

```json
{
  "formalParameter": {
    "formalParameterId": "a1b2c3d4-aaaa-1111-2222-333344440001",
    "formalParameterName": "environment",
    "procedureName": "Deploy App",
    "projectName": "Online Banking",
    "description": "Target deployment environment",
    "defaultValue": "staging",
    "required": "1",
    "type": "Select",
    "label": "Environment",
    "createTime": "2024-06-30T12:00:00.000Z"
  }
}
```

---

## 2. Install a Plugin

**POST** `/plugins`

Plugins are distributed as `.jar` files. Installation requires uploading the plugin file or specifying a URL.

### Required parameters (one of the following)

| Parameter | Type | Description |
|---|---|---|
| `file` | file | Plugin JAR file (multipart upload) |
| `url` | string | URL to download the plugin JAR from |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `doPromotion` | boolean | Automatically promote after install (default: `false`) |
| `force` | boolean | Force install even if version already exists |

### curl example — install from URL

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/plugins" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://downloads.cloudbees.com/plugins/EC-Git-2.5.0.jar",
    "doPromotion": false
  }'
```

### curl example — install from file upload

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/plugins" \
  -H "sessionid: <API-token>" \
  -F "file=@/tmp/EC-Git-2.5.0.jar"
```

### Python example

```python
import requests

# Install from URL
result = requests.post(
    f"{cdro.base}/plugins",
    headers=cdro.headers,
    json={"url": "https://downloads.cloudbees.com/plugins/EC-Git-2.5.0.jar", "doPromotion": False},
    verify=False
).json()
print(result["plugin"]["pluginName"])

# Install from file
with open("/tmp/EC-Git-2.5.0.jar", "rb") as f:
    result = requests.post(
        f"{cdro.base}/plugins",
        headers=cdro.headers,
        files={"file": f},
        verify=False
    ).json()
```

### Example JSON response

```json
{
  "plugin": {
    "pluginId": "b2c3d4e5-bbbb-2222-3333-444455550002",
    "pluginName": "EC-Git-2.5.0",
    "pluginKey": "EC-Git",
    "version": "2.5.0",
    "status": "installed",
    "promoted": "0",
    "createTime": "2024-06-30T12:10:00.000Z"
  }
}
```

---

## 3. Promote a Plugin (PUT action)

**PUT** `/plugins/{pluginName}`

Promoting a plugin makes it the active version used for new configurations. Only one version of a plugin can be promoted at a time.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `promotePlugin` |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/plugins/EC-Git-2.5.0" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{"request": "promotePlugin"}'
```

### Python example

```python
import requests

plugin = "EC-Git-2.5.0"
url = f"{cdro.base}/plugins/{plugin}"
result = requests.put(url, headers=cdro.headers,
                      json={"request": "promotePlugin"}, verify=False).json()
print(result["plugin"]["promoted"])
```

### Example JSON response

```json
{
  "plugin": {
    "pluginId": "b2c3d4e5-bbbb-2222-3333-444455550002",
    "pluginName": "EC-Git-2.5.0",
    "pluginKey": "EC-Git",
    "version": "2.5.0",
    "promoted": "1",
    "promotedTime": "2024-06-30T12:15:00.000Z"
  }
}
```

---

## 4. Create an Email Configuration

**POST** `/emailConfigs`

Email configurations define SMTP server settings for sending notifications.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `emailConfigName` | string | Name of the email configuration |
| `mailHost` | string | SMTP server hostname |

### Optional parameters

| Parameter | Type | Description |
|---|---|---|
| `mailPort` | integer | SMTP port (default: `25`) |
| `mailUser` | string | SMTP authentication username |
| `mailPassword` | string | SMTP authentication password |
| `mailFrom` | string | Default sender address |
| `mailProtocol` | string | `SMTP`, `SMTPS`, or `SMTP_TLS` |
| `description` | string | Configuration description |

### curl example

```bash
curl -k -X POST \
  "https://cdro.example.com/rest/v1.0/emailConfigs" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "emailConfigName": "corporate-smtp",
    "mailHost": "smtp.example.com",
    "mailPort": 587,
    "mailUser": "cdro-notifications@example.com",
    "mailPassword": "smtpP@ssword!",
    "mailFrom": "cdro-notifications@example.com",
    "mailProtocol": "SMTP_TLS",
    "description": "Corporate SMTP relay"
  }'
```

### Python example

```python
result = cdro.post(
    "emailConfigs",
    emailConfigName="corporate-smtp",
    mailHost="smtp.example.com",
    mailPort=587,
    mailUser="cdro-notifications@example.com",
    mailPassword="smtpP@ssword!",
    mailFrom="cdro-notifications@example.com",
    mailProtocol="SMTP_TLS",
    description="Corporate SMTP relay"
)
print(result["emailConfig"]["emailConfigId"])
```

### Example JSON response

```json
{
  "emailConfig": {
    "emailConfigId": "c3d4e5f6-cccc-3333-4444-555566660003",
    "emailConfigName": "corporate-smtp",
    "mailHost": "smtp.example.com",
    "mailPort": "587",
    "mailFrom": "cdro-notifications@example.com",
    "mailProtocol": "SMTP_TLS",
    "description": "Corporate SMTP relay",
    "createTime": "2024-06-30T12:20:00.000Z"
  }
}
```

---

## 5. Send a Test Email (PUT action)

**PUT** `/emailConfigs/{configName}`

Tests an email configuration by sending a test message.

### Required parameters

| Parameter | Type | Description |
|---|---|---|
| `request` | string | Must be `testEmailConfig` |
| `mailTo` | string | Recipient address for the test email |

### curl example

```bash
curl -k -X PUT \
  "https://cdro.example.com/rest/v1.0/emailConfigs/corporate-smtp" \
  -H "sessionid: <API-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "request": "testEmailConfig",
    "mailTo": "admin@example.com"
  }'
```

### Python example

```python
import requests

url = f"{cdro.base}/emailConfigs/corporate-smtp"
result = requests.put(url, headers=cdro.headers, json={
    "request": "testEmailConfig",
    "mailTo": "admin@example.com"
}, verify=False).json()
print("Test email sent:", result)
```

### Example JSON response

```json
{
  "emailConfig": {
    "emailConfigName": "corporate-smtp",
    "testStatus": "success",
    "message": "Test email sent successfully to admin@example.com"
  }
}
```

---

## Notes

- **Parameter types:** The `Select` and `Checkbox` types can be paired with `optionsDSL` to dynamically populate available choices at runtime from a property sheet or procedure call.
- **Required vs optional parameters:** Setting `required: true` on a formal parameter causes the UI and API to enforce that a value is provided. Calls without the required parameter will fail with a validation error.
- **Plugin promotion:** Only one version of each plugin (by `pluginKey`) can be promoted at a time. Promoting a new version automatically demotes the old one.
- **Email in procedures:** Reference email configs in step commands or use the `EC-Email` plugin's `sendEmail` procedure to send notifications with template support.
- **SMTP password security:** SMTP passwords in POST/PUT requests are stored encrypted. Treat them as credentials — use HTTPS and avoid logging request bodies.
- **Swagger UI:** Full parameter reference including pipeline formal parameters and plugin management options at `https://<cdro-server>/rest/doc/v1.0/`.