# Claude Code Skills

A collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) / [oh-my-claudecode](https://github.com/oh-my-claudecode/oh-my-claudecode) skills for software engineering workflows.

## Skills

### `cloudbees_cdro_api`

A comprehensive reference assistant for the [CloudBees CD/RO](https://www.cloudbees.com/products/cd-ro) (formerly Electric Cloud ElectricFlow) API.

Covers all 5 API interfaces:

| API | Usage |
|-----|-------|
| **ectool CLI** | `ectool commandName [args] [--flags]` |
| **Perl** | `$cmdr->commandName(named => val)` |
| **Groovy** | `ef.commandName(named: val)` |
| **DSL** | `ectool evalDsl --dslFile script.dsl` |
| **REST** | curl / Python / any HTTP client |

Organized into 9 category groups × 5 APIs = 45 reference files:

| Category | Key Commands |
|----------|-------------|
| `project-procedure-step` | createProject, createProcedure, createStep, runProcedure |
| `job-property` | getJob, abortJob, waitForJob, setProperty, getProperty |
| `pipeline-release-stage` | createPipeline, runPipeline, createRelease, startRelease, createStage, createTask |
| `application-component` | createApplication, deployApplication, createComponent, createProcess |
| `resource-environment` | createResource, createEnvironment, provisionEnvironment, pingResource |
| `credential-acl-user` | createCredential, createUser, createGroup, createAclEntry |
| `schedule-artifact` | createSchedule, publishArtifactVersion, createSnapshot, createTag |
| `parameter-plugin-email` | createFormalParameter, installPlugin, promotePlugin, sendEmail |
| `workspace-workflow-catalog-server` | createWorkspace, createWorkflowDefinition, createTrigger, evalDsl, findObjects |

#### Installation

Copy `cloudbees_cdro_api/` to your Claude Code skills directory:

```bash
cp -r cloudbees_cdro_api ~/.claude/skills/cloudbees_cdro_api
```

Or with oh-my-claudecode:

```bash
cp -r cloudbees_cdro_api ~/.claude/skills/cloudbees_cdro_api
```

Then invoke in any Claude Code session:

```
/cloudbees_cdro_api
```

#### Official Docs

- Perl/ectool: https://docs.cloudbees.com/apidocs/ec-perl/latest
- Groovy: https://docs.cloudbees.com/apidocs/ec-groovy/latest
- DSL: https://docs.cloudbees.com/apidocs/dsl/latest
- REST: https://docs.cloudbees.com/docs/cloudbees-cd-api-rest/latest/

## License

MIT
