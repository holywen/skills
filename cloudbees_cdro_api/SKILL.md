---
name: cloudbees_cdro_api
description: >
  CloudBees CD/RO full API assistant: Perl (ec-perl/ectool), Groovy (ec-groovy),
  DSL (Groovy/YAML), and REST API. Read the relevant category file on demand
  for detailed commands, parameters, and examples. Never fetch web docs unless
  the category file doesn't cover the specific command.
user-invocable: true
---

# CloudBees CD/RO — Full API Reference Assistant

## How to answer questions

**Step 1 — Identify the API and category** from the user's question.

**Step 2 — Read the relevant category file** using the Read tool. Each file is
detailed (~150-200 lines) with command tables, parameter descriptions, and
realistic code examples.

**Step 3 — If the specific command is not in the file**, fetch its official doc:
- Perl/ectool: `https://docs.cloudbees.com/apidocs/ec-perl/latest/c/{commandName}`
- Groovy: `https://docs.cloudbees.com/apidocs/ec-groovy/latest/c/{commandName}`
- DSL: `https://docs.cloudbees.com/apidocs/dsl/latest/c/{objectName}`
- REST guide: `https://docs.cloudbees.com/docs/cloudbees-cd-api-rest/latest/`

---

## Category File Map

Base path: `~/.claude/skills/ectool-cdro-api/categories/{api}/{category}.md`

| Category | Covers |
|----------|--------|
| `project-procedure-step` | createProject, createProcedure, createStep, runProcedure, copyProcedure |
| `job-property` | getJob, abortJob, waitForJob, getJobStepLog, setProperty, getProperty, expandString |
| `pipeline-release-stage` | createPipeline, runPipeline, abortPipelineRun, createRelease, startRelease, createStage, createTask, completeManualTask, createGate |
| `application-component` | createApplication, deployApplication, createComponent, createProcess, createProcessStep, runProcess |
| `resource-environment` | createResource, createResourcePool, pingResource, createEnvironment, createEnvironmentTier, addEnvironmentTierResource |
| `credential-acl-user` | createCredential, attachCredential, createAclEntry, getAcl, createUser, createGroup, addUserToGroup |
| `schedule-artifact` | createSchedule, runSchedule, publishArtifactVersion, retrieveArtifactVersions |
| `parameter-plugin-email` | createFormalParameter, setOutputParameter, installPlugin, promotePlugin, createPluginConfiguration, createEmailConfig, sendEmail |
| `workspace-workflow-catalog-server` | createWorkspace, createGateway, createZone, runWorkflow, createTrigger, createScmSync, createCatalog, runCatalogItem, evalDsl, findObjects, export, generateDsl |

### API subfolder by interface

| API | Subfolder | Runtime |
|-----|-----------|--------|
| Perl | `perl/` | `ec-perl script.pl` |
| ectool CLI | `ectool/` | `ectool commandName args` |
| Groovy | `groovy/` | `ec-groovy script.groovy` |
| DSL | `dsl/` | `ectool evalDsl --dslFile script.dsl` |
| REST | `rest/` | curl / Python / any HTTP client |

**Examples of which file to read:**
- "ectool로 artifact publish" → Read `categories/ectool/schedule-artifact.md`
- "ectool run pipeline 怎么写" → Read `categories/ectool/pipeline-release-stage.md`
- "Groovy로 pipeline 만들기" → Read `categories/groovy/pipeline-release-stage.md`
- "REST API로 job 상태 확인" → Read `categories/rest/job-property.md`
- "DSL로 application 정의" → Read `categories/dsl/application-component.md`
- "Perl에서 credential attach" → Read `categories/perl/credential-acl-user.md`

---

## Quick Connection Reference

### Perl
```perl
use ElectricCommander;
my $cmdr = new ElectricCommander();   # uses ELECTRIC_COMMANDER_SERVER
```

### Groovy
```groovy
import com.electriccloud.client.groovy.ElectricFlow
ElectricFlow ef = new ElectricFlow()  // uses COMMANDER_SERVER
```
```bash
ec-groovy -DCOMMANDER_SERVER=cdro.example.com -DCOMMANDER_SECURE=1 script.groovy
```

### DSL
```bash
ectool evalDsl --dslFile script.dsl          # Groovy DSL
ectool evalDsl --format yaml --dslFile s.yaml # YAML DSL
```

### REST
```bash
ectool login admin changeme --server cdro.example.com   # get sessionId
curl -k -X GET "https://<server>/rest/v1.0/projects" \
     -H "sessionid: <API-token>"
# Swagger UI: https://<server>/rest/doc/v1.0/
```

### Environment Variables

| Variable | API | Purpose |
|----------|-----|---------|
| `ELECTRIC_COMMANDER_SERVER` | Perl/ectool | Server hostname |
| `COMMANDER_SERVER` | Groovy | Server hostname |
| `ELECTRIC_COMMANDER_PORT` | Perl/ectool | Port (default 8443) |
| `COMMANDER_SESSIONID` | Perl/ectool | Session token |
