---
title: "TEST"
header:
author: Tom Degreef
date: 2000-12-30
categories:

tags:
---


# test text 3 #

{% capture code %}
``` powershell
Import-Module azure.storage

if (!(test-path "C:\LogsFromAzure"))
{
    New-Item -ItemType Directory -Path "C:\LogsFromAzure"
}

$BlobProperties = @{

    StorageAccountName   = 'osdlogging'
    storSas              = '?sv=2019-12-12<<<rest of key is censored>>>'
}

$clientContext = New-AzureStorageContext -SasToken ($BlobProperties.storsas) -StorageAccountName ($blobproperties.StorageAccountName)

$files = Get-AzureStorageBlob -Container "osdlogs" -Context $clientContext 

foreach ($file in $files)
{
    write-host $file.name
    Get-AzureStorageBlobContent -Destination "C:\LogsFromAzure" -Container "osdlogs" -Context $clientContext -Blob $file.name
   
    write-host "expanding logfiles"
    Expand-Archive -Path "C:\LogsFromAzure\$($file.name)" -DestinationPath "C:\LogsFromAzure"

    write-host "cleaning up"
    Remove-Item -Path "C:\LogsFromAzure\$($file.name)" -Force

    Remove-AzureStorageBlob -Container "osdlogs" -Context $clientContext -Blob $file.name
  
}
```
{% endcapture %}
{% include code.html code=code lang="powershell" %}