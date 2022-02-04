# SQL MI BYOK ARM + API

### References

**ARM**
* https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/managedinstances/keys?tabs=json
* https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/managedinstances/encryptionprotector?tabs=json

**API**
* https://docs.microsoft.com/en-us/rest/api/sql/2021-05-01-preview/managed-instance-keys
* https://docs.microsoft.com/en-us/rest/api/sql/2021-05-01-preview/managed-instance-encryption-protectors


### Sample Powershell script

``` Powershell
#########################
# Deploy via ARM template
#########################
# Deploys Key, then Encryptor
az deployment group create `
    --resource-group aia-sqlinsights-rg `
    --name mibyok1 `
    --template-file .\template.json

# Docs:
# https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/managedinstances/keys?tabs=json
# https://docs.microsoft.com/en-us/azure/templates/microsoft.sql/managedinstances/encryptionprotector?tabs=json

#########################
# Key CRUDs via API
#########################
# Docs:
# https://docs.microsoft.com/en-us/rest/api/sql/2021-05-01-preview/managed-instance-keys

# Get OAuth Token first
# List By Keys
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/keys?api-version=2021-05-01-preview' -Method 'GET' -Headers $headers
$response | ConvertTo-Json

# Get Particular Key
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/keys/my-prod-akv_PROD-TDE_2c7e882ad0a44dc9850c1ea3824f98cb?api-version=2021-05-01-preview' -Method 'GET' -Headers $headers
$response | ConvertTo-Json

# Create or Update Key
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")
$headers.Add("Content-Type", "application/json")

$body = "{
`n  `"properties`": {
`n    `"serverKeyType`": `"AzureKeyVault`",
`n    `"uri`": `"https://my-prod-akv.vault.azure.net/keys/PROD-TDE/2c7e882ad0a44dc9850c1ea3824f98cb`"
`n  }
`n}"

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/keys/my-prod-akv_PROD-TDE_2c7e882ad0a44dc9850c1ea3824f98cb?api-version=2021-05-01-preview' -Method 'PUT' -Headers $headers -Body $body
$response | ConvertTo-Json

# Delete Particular Key
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/keys/my-prod-akv_PROD-TDE_2c7e882ad0a44dc9850c1ea3824f98cb?api-version=2021-05-01-preview' -Method 'DELETE' -Headers $headers
$response | ConvertTo-Json

#########################
# Rotations via API
#########################
# Docs
# https://docs.microsoft.com/en-us/rest/api/sql/2021-05-01-preview/managed-instance-encryption-protectors

# List All Protectors
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/encryptionProtector?api-version=2021-05-01-preview' -Method 'GET' -Headers $headers
$response | ConvertTo-Json

# Get Current Protector
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/encryptionProtector/current?api-version=2021-05-01-preview' -Method 'GET' -Headers $headers
$response | ConvertTo-Json

# Revalidate
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/encryptionProtector/current/revalidate?api-version=2021-05-01-preview' -Method 'POST' -Headers $headers
$response | ConvertTo-Json

# Update Protector to AKV
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")
$headers.Add("Content-Type", "application/json")

$body = "{
`n  `"properties`": {
`n    `"serverKeyType`": `"AzureKeyVault`",
`n    `"serverKeyName`": `"my-prod-akv_PROD-TDE_2c7e882ad0a44dc9850c1ea3824f98cb`",
`n    `"autoRotationEnabled`": false
`n  }
`n}"

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/encryptionProtector/current?api-version=2021-05-01-preview' -Method 'PUT' -Headers $headers -Body $body
$response | ConvertTo-Json

# Update Protector to Managed
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization", "Bearer INSERT_BEARER_TOKEN")
$headers.Add("Content-Type", "application/json")

$body = "{
`n  `"properties`": {
`n    `"serverKeyType`": `"ServiceManaged`",
`n    `"serverKeyName`": `"ServiceManaged`"
`n  }
`n}"

$response = Invoke-RestMethod 'https://management.azure.com/subscriptions/YOUR--SUBSCRIPTION--ID/resourceGroups/aia-sqlinsights-rg/providers/Microsoft.Sql/managedInstances/aiasqlmisvr01/encryptionProtector/current?api-version=2021-05-01-preview' -Method 'PUT' -Headers $headers -Body $body
$response | ConvertTo-Json
```