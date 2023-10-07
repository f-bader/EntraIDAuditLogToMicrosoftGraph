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

Only time will tell.
