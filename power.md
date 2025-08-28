```ps
# Variables
$ClientId     = "YOUR_CLIENT_ID"
$ClientSecret = "YOUR_CLIENT_SECRET"
$AccountUrn   = "urn:dtaccount:YOUR_ACCOUNT_UUID"
$Scopes       = "storage:buckets:write storage:buckets:read storage:logs:write storage:logs:read app-engine:apps:run"

# Token request body
$Body = @{
    grant_type    = "client_credentials"
    client_id     = $ClientId
    client_secret = $ClientSecret
    scope         = $Scopes
    resource      = $AccountUrn
}

# Request token
$Response = Invoke-RestMethod -Method Post `
    -Uri "https://sso.dynatrace.com/sso/oauth2/token" `
    -ContentType "application/x-www-form-urlencoded" `
    -Body $Body

# Extract token
$AccessToken = $Response.access_token
Write-Host "Access Token: $AccessToken"

```


other script

```
# API endpoint
$UploadUrl = "https://cfc52677.apps.dynatrace.com/platform/storage/resource-store/v1/files/tabular/lookup:upload"

# Request metadata (lookup definition JSON)
$RequestJson = @{
    lookupField   = "id"
    filePath      = "/lookups/mydata"
    overwrite     = $false
    displayName   = "My lookup data"
    skippedRecords= 0
    autoFlatten   = $true
    timezone      = "UTC"
    locale        = "en_US"
    description   = "Description of my lookup data"
    parsePattern  = "LD:id ',' LD:value"
} | ConvertTo-Json -Compress

# Multipart form data
$Form = @{
    request = $RequestJson
    content = Get-Item "C:\path\to\table-data.csv"
}

# Upload request
Invoke-RestMethod -Method Post `
    -Uri $UploadUrl `
    -Headers @{ Authorization = "Bearer $AccessToken" } `
    -Form $Form
```
