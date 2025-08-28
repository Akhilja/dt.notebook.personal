```ps
# ------------------------
# 1️⃣ GET OAUTH TOKEN
# ------------------------
$ClientId     = "dt0s02.DAFW6AVJ"
$ClientSecret = "dt0s02.DAFW6AVJ.VGQQVRNDURWTQKDLY6TVDSDUTMT432VTRUJG3TKW7UUMNZOWTSJOMPFQJDGHQXQY"
$AccountUrn   = "urn:dtaccount:30af6f9a-9fd9-4da3-9bf6-b3321b331a64"
$Scopes       = "storage:files:read storage:files:write storage:files:delete"

function Encode([string]$s) { [System.Net.WebUtility]::UrlEncode($s) }

$Body = [string](
    "grant_type=client_credentials" +
    "&client_id=" + (Encode $ClientId) +
    "&client_secret=" + (Encode $ClientSecret) +
    "&scope=" + (Encode $Scopes) +
    "&resource=" + (Encode $AccountUrn)
)

try {
    $TokenResponse = Invoke-RestMethod -Method Post `
        -Uri "https://sso.dynatrace.com/sso/oauth2/token" `
        -ContentType "application/x-www-form-urlencoded" `
        -Body $Body

    $AccessToken = $TokenResponse.access_token.Trim('"').Trim()
    Write-Host "✅ Access Token retrieved successfully"
} catch {
    Write-Error "❌ Failed to get OAuth token: $_"
    exit 1
}

# ------------------------
# 2️⃣ UPLOAD LOOKUP CSV
# ------------------------
$UploadUrl = "https://cfc52677.apps.dynatrace.com/platform/storage/resource-store/v1/files/tabular/lookup:upload"
$CsvPath = "C:\path\to\table-data.csv"  # Windows
# $CsvPath = "/Users/akhil_jayendran/Downloads/table-data.csv"  # macOS

$RequestJson = @{
    lookupField    = "id"
    filePath       = "/lookups/mydata"
    overwrite      = $false
    displayName    = "My lookup data"
    skippedRecords = 0
    autoFlatten    = $true
    timezone       = "UTC"
    locale         = "en_US"
    description    = "Description of my lookup data"
    parsePattern   = "LD:id ',' LD:value"
} | ConvertTo-Json -Compress

$Form = @{
    request = $RequestJson
    content = Get-Item $CsvPath
}

try {
    $UploadResponse = Invoke-RestMethod -Method Post `
        -Uri $UploadUrl `
        -Headers @{ Authorization = "Bearer $AccessToken"; Accept = "*/*" } `
        -Form $Form

    Write-Host "✅ Lookup file uploaded successfully"
    Write-Host ($UploadResponse | ConvertTo-Json -Depth 5)
} catch {
    Write-Error "❌ Failed to upload lookup file: $_"
}

```


```bash
 TOKEN=$(curl -s -X POST "https://sso.dynatrace.com/sso/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=dt0s02.DAFW6AVJ" \
  -d "client_secret=dt0s02.DAFW6AVJ.VGQQVRNDURWTQKDLY6TVDSDUTMT432VTRUJG3TKW7UUMNZOWTSJOMPFQJDGHQXQY" \
  -d "scope=storage:files:read storage:files:write storage:files:delete" \
  -d "resource=urn:dtaccount:30af6f9a-9fd9-4da3-9bf6-b3321b331a64"\
   | jq -r '.access_token')
```
