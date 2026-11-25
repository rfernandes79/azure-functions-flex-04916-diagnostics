# Evidence scope and redaction notes

## Provenance

The operator supplied Azure portal/Cloud Shell summaries of ARM queries and an Application Insights trace export. This repository is a curated, sanitized rendering of those observations. The publishing assistant did not execute the Azure deployment, query the live Azure resources directly, or independently sign the received evidence.

The deployment pipeline is dated 2026-11-24 in UTC. The current configuration/IAM summary is dated 2026-11-25T22:11:58.107145+00:00. Current configuration does not prove what every instance used during the earlier deployment. The exact order of each authenticated HTTP test relative to the transition to RuntimeFailed has not been established.

## Pipeline logs

The received diagnostic had 51 reported rows and 45 distinct events across seven tables, with no truncation marker on the received tables. The public pipeline log retains all 45 distinct events and removes exact duplicates. It is not an assertion that all platform telemetry was captured. The detector's field named `exception` is reproduced as a label; values such as `Specialize` or `Save` are not independently diagnosed application exceptions. Pod-deletion/cleanup messages do not establish the timeout cause.

## Host trace sample

The received trace excerpt contains 108 Host.Startup Information records across nine role-instance identifiers. Eight complete sequences list the 18 expected functions; the first received instance has only the tail of startup. The public sample is one complete sequence of 13 records. Its 18 function names were replaced with consistent aliases. Matching names is not a per-process byte attestation.

The original export header, executed query hash and total/truncation state were unavailable. Sampling was enabled and ItemCount was not projected. Counts represent observed rows. Blank exception fields and absent worker records in this excerpt do not establish an error-free observation window. Informational host-stop events do not prove a crash. The Custom-provider zero-functions line must be read with the following 18-functions-loaded line.

## Redaction

- Application name and hostname: `[APP_NAME]`, `[APP_HOSTNAME]`.
- Deployment and correlation identifiers: `[DEPLOYMENT_ID]`, `[CORRELATION_ID]`.
- Earlier deployment referenced by automatic cleanup: `[PREVIOUS_DEPLOYMENT_ID]`.
- Other opaque identifiers: `[IDENTIFIER_nn]`, consistently within the exported logs.
- Function names: `Host.Functions.function_nn`, representing names that were present in the received logs.
- Existing path suppression: `[PREVIOUSLY_REDACTED_PATH]`; its original value was not recovered.

The error wording is preserved apart from the named identifier replacements. No business data, source workbooks, private maps, private repository URLs, Graph identifiers, access tokens, connection strings or signed URLs were selected for this public repository. A private alias map is kept outside this directory and is not part of the public files.

## Interpretation limits

The package reached pipeline completion, tested authenticated application paths responded, and a later deployment-status response reported RuntimeFailed/ExpectationFailed/04916. These observations do not determine all-instance health, effective access, allocation failure or the underlying controller condition.

The documentation describes validation on a temporary instance and a possible fallback to an earlier deployment process after validation failure/timeout. The observed fallback path has not been independently established here. The related public issue11952 was closed following platform-deployment triage; its two comments did not publish a root cause or fix. No workaround, runtime change, permission grant, firewall change or new deployment is recommended by this repository without a specific evidenced reason.
