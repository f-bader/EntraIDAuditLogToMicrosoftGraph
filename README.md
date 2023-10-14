# Entra ID Audit Log To Microsoft Graph Request Uri

A list of Entra ID (Azure AD) Audit event names and the corresponding Microsoft Graph Request Uri

The data is provided as CSV and JSON for your convenience.

* [AuditEventToGraphUri.json](./output/AuditEventToGraphUri.json)
* [AuditEventToGraphUri.csv](./output/AuditEventToGraphUri.csv)

```csv
EntraIDOperationName,MicrosoftGraphRequestUri,RequestMethod,EntraIDOperationVersion
"Add conditional access policy","https:/graph.microsoft.com/<MSGraphVersion>/identity/conditionalAccess/policies",POST,"1.0"
"Delete FIDO2 security key(s)","https:/graph.microsoft.com/<MSGraphVersion>/users/<UUID>/authentication/fido2Methods/<ID>",DELETE,"1.0"
```

## How to contribute?

Please create a pull request with your own CSV added to the source folder.

## FAQ

### How to create the CSV?

You must enable the [Microsoft Graph activity logs](https://learn.microsoft.com/en-us/graph/microsoft-graph-activity-logs-overview) in your Entra ID and forward the logs to a Log Analytics workspace. After a few days you can use the query below to generate the file in the correct format. Please create a pull request and put the file in the source directory. This way we can built a large dataset of Entra ID Activity logs to Microsoft Graph URI mappings.

```kql
AuditLogs
| where TimeGenerated > ago(90d)
| join kind=inner (
    MicrosoftGraphActivityLogs
    // Ignore GET requests
    | where RequestMethod != 'GET'
    )
    on $left.CorrelationId == $right.ClientRequestId
// Remove PII information and normalize the RequestURI
| extend NormalizedRequestUri = replace_regex(RequestUri, @'[0-9a-fA-F]{8}\b-[0-9a-fA-F]{4}\b-[0-9a-fA-F]{4}\b-[0-9a-fA-F]{4}\b-[0-9a-fA-F]{12}', @'<UUID>')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'[a-zA-Z0-9_-]{41,65}', @'<ID>')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'\d+$', @'<ID>')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'\/+', @'/')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'https:\/', @'https://')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'%23EXT%23', @'')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'\/[a-zA-Z0-9+_.\-]+@[a-zA-Z0-9.]+\/', @'/<UPN>/')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'^\/<UUID>', @'')
| extend NormalizedRequestUri = replace_regex(NormalizedRequestUri, @'\?.*$', @'')
// Remove POST requests to the batch endpoint
| where not ( NormalizedRequestUri matches regex @"https:\/\/graph.microsoft.com\/(v1\.0|beta)/\$batch" )
| summarize by OperationName, NormalizedRequestUri, RequestMethod, OperationVersion
| project-rename
    MicrosoftGraphRequestUri = NormalizedRequestUri,
    EntraIDOperationName = OperationName,
    EntraIDOperationVersion = OperationVersion
| sort by EntraIDOperationName asc 
```
