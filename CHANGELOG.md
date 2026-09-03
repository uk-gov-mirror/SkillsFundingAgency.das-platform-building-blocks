# 3.3.0

DASD-15605: Extended `log-analytics-workspace.json` to support security and audit workspaces. Added optional `totalRetentionInDays` and `tableNames` to configure long term (archive) retention per table, `dailyQuotaGb`, `disableLocalAuth`, `enableDataExport` and `tags`. Bumped the workspace apiVersion from `2020-08-01` to `2025-02-01`, which is required because `disableLocalAuth` and `enableDataExport` were only added to `WorkspaceFeatures` in `2020-10-01`, and table level retention needs `2022-10-01` or later.

Backward compatible. Every new parameter defaults to the existing behaviour: `disableLocalAuth` and `enableDataExport` default to `false`, `tableNames` is empty so no table resources are deployed, and `tags` and `dailyQuotaGb` are omitted from the request entirely when not supplied so that tags and daily caps set outside this template are not reset. The `dataRetentionDays` `maxValue` was widened from 90 to 730; its default is unchanged at 31. Both existing outputs (`resourceId`, `fullyQualifiedResourceId`) are unchanged. Verified with `what-if` against an existing workspace using the minimal caller shape (name and sku only), which reported `NoChange`.

Added `log-analytics-workspace-data-export.json` to deploy a `Microsoft.OperationalInsights/workspaces/dataExports` rule, streaming whole tables to a storage account or Event Hubs namespace as the data arrives. Data export cannot filter rows, so the selection is a table list. Deploys nothing when `tableNames` is empty or no destination is supplied. The workspace must be deployed with `enableDataExport` set to `true`.

Added `storage-account-immutable.json` to deploy a storage account with a WORM (write once, read many) immutability policy, for tamper protected retention of exported audit and security logs. `immutabilityPolicyState` accepts `Disabled`, `Unlocked` or `Locked`. **`Locked` is irreversible** - the blobs, container, storage account and its resource group cannot be deleted until the immutability period expires, and this cannot be overridden by subscription administrators or Microsoft Support. A policy can only be created `Disabled` or `Unlocked`; only an `Unlocked` policy can transition to `Locked`, so roll out `Unlocked` first and lock as a second, approved deployment. This is a separate template rather than a change to `storage-account-arm.json` because account level immutability can only be set at account creation time and so cannot be applied to existing accounts.

# 3.2.1

DASD-15508: Fixed `scheduled-query-alert.json` so the `throttlingInMin` (action suppression) property is omitted entirely when `autoMitigate` is `true`. Azure rejects a scheduledQueryRule that has both auto-mitigation and action suppression set ("Auto mitigation must be disabled when action suppression is set"), and setting the throttle to 0 is not sufficient - the property must be absent. The action object is now built with `union()` so alerts that do not opt into `autoMitigate` keep the previous 20 minute throttle, leaving existing behaviour unchanged.

# 3.2.0

DASD-15509: Added `logic-app.json` to support deploying Azure Logic Apps (`Microsoft.Logic/workflows`) with a caller-supplied workflow definition and parameters and an optional system-assigned managed identity. Outputs the Logic App resource id and identity principalId.

Added an optional `additionalActionGroupIds` array parameter to `scheduled-query-alert.json` so an alert can invoke extra action groups (e.g. one that triggers a remediation Logic App) alongside the shared notification group passed as `actionGroupId`. Backward compatible (defaults to an empty array).

Corrected the `allowedValues` for the `alertTriggerOperator` parameter in `scheduled-query-alert.json` to the operators actually supported by the scheduledQueryRules metricTrigger (`Equal`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`). The previous list only permitted `GreaterThan` and the invalid value `EqualTo`, which blocked alerts that need `GreaterThanOrEqual`. Existing callers pass `GreaterThan`, which is unaffected.

Added an optional `autoMitigate` boolean parameter to `scheduled-query-alert.json` so an alert can be automatically resolved (sending a resolved notification to its action groups) once the query stops breaching. Backward compatible (defaults to `false`, preserving the existing fire-only behaviour).

# 3.1.0

DASD-12494: CDN migration from Edgio to Azure Front Door. Added `afd-profile.json` and `afd-endpoint.json` to support Azure Front Door resources.

# 3.0.0

This change allows the SQL server password to be fetched during pipeline runtime to allow easier rotation of the secret.

Breaking change to [sql-dacpac-deploy.yml](azure-pipelines-templates/deploy/step/sql-dacpac-deploy.yml). Removed ```${{ parameters.SqlPassword }}``` which is passed into template from service pipelines, no longer required when using AZ CLI. Any pipeline using this template will need to remove the parameter before bumping to this version.

# 2.2.13

Added applicaton insights failed request template and get product app insights infomation step

# 2.2.3

Setting default value of WEBSITE_ADD_SITENAME_BINDINGS_IN_APPHOST_CONFIG to 1

# 2.2.0

Added encrypyion at host property on AKS and AKS node pools

# 2.1.26

Adding the health check property to app-service-v2.json

# 2.1.22

Addition to the role-assignments.json, created a template for Log Analytics role assignment

# 2.1.21

Updated SonarCloud config to use the latest version currently 2.x

# 2.1.1

Removed unused azure-pipelines-templates/deploy/job/arm-deploy.yml and moved placeholder file.

# 2.1.0

templates/cosmos-db.json: Set backupPolicy to Continuous mode (30 days) for Azure Cosmos DB accounts.

# 2.0.5

Migrated from azure-pipelines.yml to .github/workflows/release.yml to avoid PAT token usage.

Addition to the app-build.yml azure-pipelines-template to comment on a Pull Request if the Package Scanning step detects any vulnerabilities.

# 2.0.3

Addition of two azure-pipelines-templates to allow app services including function apps to whitelist and remove the whilist of the pipeline agents.

Needed for automation test suite pipelines.

# 2.0.1

Two new building block templates added to allow pipelines to be whitelisted on a singular or multiple app services and then be removed again.

Addition of the following files -

[appservice-whitelist-ip.yml](https://github.com/SkillsFundingAgency/das-platform-building-blocks/tree/master/azure-pipelines-templates/deploy/step/appservice-whitelist-ip.yml)

[appservice-remove-ip.yml](https://github.com/SkillsFundingAgency/das-platform-building-blocks/tree/master/azure-pipelines-templates/deploy/step/appservice-remove-ip.yml)

# 2.0.0

Breaking change to [generate-config.yml](azure-pipelines-templates/deploy/step/generate-config.yml).  Additional parameters have been added to this step template and changes have been made to the way secrets are handled.  When updating to version 2 or beyond you will need to add EnvironmentName, StorageAccountName & StorageResourceGroup inputs.  You will also need to map any secret variables that are used in config generation in the ConfigurationSecrets variable.  Typically your call to the step template should look something like:
```
- template: azure-pipelines-templates/deploy/step/generate-config.yml@das-platform-building-blocks
  parameters:
    EnvironmentName: $(EnvironmentName)
    ServiceConnection: ${{ parameters.ServiceConnection }}
    SourcePath: $(Pipeline.Workspace)/das-employer-config/Configuration/das-foo-web
    StorageAccountName: $(ConfigurationStorageAccountName)
    StorageAccountResourceGroup: $(SharedEnvResourceGroup)
    ConfigurationSecrets:
      FooSecretConnectionString: $(FooSecretConnectionString)
      BarSecretKey: $(BarSecretKey)
```

# 1.0.8

Start of changelog
