# Azure Functions Flex Consumption: RuntimeFailed / 04916

Sanitized technical case report for a Flex Consumption deployment that timed out after 20 minutes while tested application paths remained available. The repository contains deployment observations, configuration summaries and redacted logs for community investigation. It is a documentation case report, not a runnable minimal reproduction.

[Resumo em português](README.pt-BR.md) · [Evidence and redaction notes](EVIDENCE.md)

## Complete error text with resource identifiers replaced

```text
Deployment for site '[APP_NAME]' with DeploymentId '[DEPLOYMENT_ID]' failed because the operation timed out after '20' minutes. Please try the deployment again.
```

`[APP_NAME]` and `[DEPLOYMENT_ID]` replace private identifiers. The remaining error wording is preserved. The retry recommendation is part of the received message; no retry was performed for this investigation.

## Deployment command used

The operator used a Python orchestration script in Azure Cloud Shell Bash to call Azure CLI. The following reference command uses placeholders throughout:

```bash
az functionapp deployment source config-zip \
  --subscription 'SUBSCRIPTION_ID' \
  --resource-group 'RESOURCE_GROUP' \
  --name 'APP_NAME' \
  --src 'PATH_TO_PINNED_ZIP' \
  --build-remote false \
  --timeout 90 \
  --only-show-errors \
  --output json
```

The CLI timeout setting and the later 20-minute Azure deployment-validation timeout are separate observations. The reported publication used Azure CLI; it was not an Actions/DevOps/VS Code/IaC publication.

## Included evidence

- [Deployment status and error](evidence/deployment-status.json).
- [Runtime, region, VNet, identity and Storage assignment summary](evidence/configuration-summary.json).
- [45 distinct Kudu/FDM pipeline events](logs/deployment-kudu-fdm.log), with UTC timestamps.
- [One host-startup sequence](logs/host-startup.sample.log) from a partial export; all function names are aliases.

## Case description and questions

Environment
Azure Functions Flex Consumption, Linux, Node.js 22, Brazil South, Functions host 4.x, HTTP functions using function.json. Existing authentication and managed identity were preserved.

Private endpoints for the app/Storage and IP/SCM access restrictions were not verified by these checks. Earlier audits reported required Entra authentication, Return401 for unauthenticated requests and HTTPS. Inbound authentication and network access rules are separate controls.

Current read-only configuration and IAM checks
ARM returned the VNet subnet-reference field with no configured subnet reference. The host storage and deployment storage select the same user-assigned managed identity, and both references match an identity assigned to the app. Both use the same storage account.

The host uses AzureWebJobsStorage__credential=managedidentity and a configured AzureWebJobsStorage__clientId, with Blob/Queue/Table service URI settings. There is no direct AzureWebJobsStorage connection string or managedIdentityResourceId setting.

The selected identity has Storage Blob Data Owner directly at the host storage-account scope. At the deployment-container scope, the same identity has a direct Storage Blob Data Contributor assignment and inherits Storage Blob Data Owner. The reported assignments have no conditions. Storage publicNetworkAccess is Enabled and networkAcls.defaultAction is Allow.

These are current configuration and assignment checks, not a connectivity test, an effective-permission evaluation including deny/PIM/group memberships, or proof of every instance's configuration during the earlier deployment. All 18 packaged functions have HTTP triggers; no new role assignments, firewall changes or runtime changes were made.

Deployment and observed result
We performed one controlled ZIP deployment with Azure CLI and remote build disabled. The package was independently compared by file hash. The deployment workflow did not issue stop, restart, rollback, or upload retries.

The deployment diagnostic reports package validation, extraction, post-build and storage upload completed, followed by the FDM deployment pipeline completion message. The deployment status API, version 2025-05-01, first reported BuildSuccessful and subsequently RuntimeFailed with ExpectationFailed / extendedCode 04916. The error message states a timeout after 20 minutes.

The deployment counters reported zero successful, failed and in-progress instances. properties.failedInstancesLogs was absent or null, not a confirmed empty array; no failed-instance log URL was returned.

Separately, the app was observed as Running. Version-specific authenticated application paths returned the expected output for the requests tested. This does not establish that every instance was healthy, and we have not established the exact ordering of each HTTP test relative to the transition to RuntimeFailed.

A partial trace export contains 108 Host.Startup Information records across nine role-instance identifiers. Eight complete sequences contain:

0 functions found (Custom)
18 functions loaded
Generating 18 job function(s)
Host initialized
Job host started

The eight lists contain the same 18 expected function names. We therefore do not interpret the Custom-provider line as zero application functions. Informational Job host stopped events are also present, without an identified shutdown cause in the received trace excerpt.

Observability limits
The trace export is partial. Application Insights sampling is enabled. The diagnostic displays MaxTelemetryItemsPerSecond=20, InitialSamplingPercentage=100, MinSamplingPercentage=0.1 and SamplingExcludedTypes=Request. We do not treat the initial 100 percent as a guarantee of full capture for the entire observation window, or a lack of observed exceptions as proof that none occurred. ItemCount was not projected in the excerpt.

A separate built-in detector grouped a future Node.js 22 end-of-support warning as Worker Failure. We have not established that this warning caused the deployment validation timeout, and have preserved the supported runtime.

Related report and repository triage
https://github.com/Azure/azure-functions-host/issues/11952 describes the same terminal status, 20-minute timeout and zero instance counters after candidate startup, including a minimal Node.js 22 control and Python controls across US regions. In a repository-member reply, mathewc wrote that this "sounds like a platform level deployment issue", outside the Functions host/runtime repository scope, and recommended a support case. The issue was closed as not_planned following that reply; the two comments provide no root-cause finding or fix.

Our deployment differs: it uses an existing app in Brazil South, the full application package, preserved authentication/managed identity and no remote build. We are citing a symptom match and prior platform-deployment triage, not claiming that both cases have a proven common cause.

Questions
1. The Flex documentation says a validation timeout can fall back to the previous deployment process. Which read-only status or trace distinguishes failed temporary-instance validation from fallback that nevertheless activates the package? We have not established whether this fallback occurred in our case.
2. What condition does RuntimeFailed / ExpectationFailed / 04916 represent here, and which read-only diagnostic can identify the validation stage or acknowledgement still pending when instance counters are zero and failedInstancesLogs is absent or null?
3. Given the related repository triage and the absence of a paid technical-support plan, is there an official community route to obtain platform-deployment triage, and how should any required identifying details be supplied privately?

We are seeking guidance through community support. No resource identifiers, tenant identifiers, app names, deployment/correlation IDs, credentials, signed URLs, application data or raw private diagnostic attachments are included in this public question.

References
https://github.com/Azure/azure-functions-host/issues/11952#issuecomment-5498083834
https://learn.microsoft.com/en-us/rest/api/appservice/web-apps/get-production-site-deployment-status?view=rest-appservice-2025-05-01
https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan#deployment-validation-for-code-package-apps
https://learn.microsoft.com/en-us/azure/azure-functions/configure-monitoring#configure-sampling
