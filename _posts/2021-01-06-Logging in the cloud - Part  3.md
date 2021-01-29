---
title: "Logging in a cloud-world - Part 3 (Intune/Autopilot)"
header:
author: Tom Degreef
date: 2000-01-11
categories:
  - Autopilot

tags:
  - SCCM
  - Configuration Manager
  - Endpoint Manager
  - OSD
  - Logs
  - Azure
  - Encryption
  - Intune
  - Autopilot
---

Let's implement the functionality we discussed in [Part 1](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-1/) and [Part 2](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-2/) in Autopilot/Intune.

## Intro ##

So here we are, in the 3rd part of a 2 part blog series ;-)  
After getting the second part up & running, I figured it might be nice to apply the same resolution in an autopilot scenario. This comes with its own set of challenges ofcorse, so this post aims to tackle all those challenges.

We will gather all the relevant logs at the end of an autopilot enrollment and upload them with encryption to our azure blob storage.  
We cannot simply copy/paste the SCCM/Task sequence solution since autopilot isn't a task sequence.  There is no list of sequential steps that we can edit so that the last step gathers the logs and uploads them Azure.  Time to get a bit creative then !

**Note :** we are going to re-use binaries and scripts found in [Part 1](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-1/) and [Part 2](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-2/) , but for your convenience, I will put all necessary links and scripts here as well.

## Azure Preparations ##

We are going to use Azure blob-storage as an intermediate storage solution. We will copy the logs to blob storage and retrieve them from there for investigation. 
There are multiple ways of storing logs/files in azure, there might be even better ways than how I chose to do it.  
This is just one possibility to get you started. By all means choose the solution that you are most comfortable with.

First of all, let's prepare our blob-container to host the logfiles. 
Log-on into the Azure portal with an account that has sufficient access and navigate to "Storage Accounts". Depending on the account you used, you might already see some storage containers.  
Click the "Add" button to create a new storage container.

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs2.jpg)

Select, or create a new resource group and choose a name for your container. If needed, you can adjust the options in the other sections, but for our purpose, the defaults are good enough, so just click "Review + Create".  
After reviewing click "Create" to actually create our blob container

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs3.jpg)

If you don't have it, download and install [Microsoft Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) as it offers an easy GUI to work with our storage containers.

When you first start the tool, a wizard will pop-up asking you to connect to azure storage. Select the first option "add an azure account" and click next.

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs1.jpg)

Log in with the account that you used to create the azure blob container.  
If all is well, you should be presented with a list of subscriptions that you are entitled to. Expand the subscription you just used to create your container and the container should become visible under "Storage accounts"

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs4.jpg)

Right-click "Blob Containers" and select "Create Blob Container"

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs5.jpg)

Give it a name to your liking and click OK. In my example, i chose "osdlogs".

Right-click your newly created container and select "Get Shared access Signature" (I will refer to this as the SAS key in the remainder of this blog)

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs6.jpg)

We are going to create 2 SAS-keys. 1 with "write" access to upload the logs, and 1 with additional delete rights to clean up the container after we retrieved the logs.

For our first key, configure it with these settings :

- Start time : your current date/time
- Expiry time : I set it to 1 year in the future
- Permissions : Read,Add, Create, Write, List (not shown in screenshot)
  
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs7.jpg)

Click "Create" and Copy and save(!!) the Query string. We will need it later.

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs8.jpg)

Repeat this process and create another SAS-key, but add the "Delete" permission in this one. Again, copy and save the Query string for later use.

## Package Preparations ##

We will create an Intune win32 app that will install all the necessary components for the encryption to work and then, we will create a scheduled task that will run one time at first logon. This scheduled task will kick-off a script that gathers what we need and uploads the logs.  
Since we have no control over the order that those apps will get installed during autopilot enrollment, I had to come up with a solution that would upload the logs after everything was finished. Hence the scheduled task. 

In my lab, I used the [Powershell app depoyment toolkit](https://psappdeploytoolkit.com/) to basically do everything we need to have configured. Once that is done, we need to convert it to a format Intune understands. This is done using the [Win32 content prep tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool). As an alternative, you could use my [GUI for the PSADT](https://www.oscc.be/sccm/configmgr/powershell/posh/intune/Powershell-App-deployment-toolkit-GUI/) if you are not to familiar with both the PSADT or the content prep tool.  
That's what I will be using to set everything up.

To start, download the PSADT GUI [here](https://github.com/TomDegreef/PSADT_GUI) and extract it locally in eg, c:\temp\psadtgui  
That should give you something like this 

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs24.jpg)

Next, create a new subfolder in temp (c:\temp\sourefiles) that will hold the following items :
* Azure.Storage (Powershell module)
* AzureRM.Profile (powershell module)
* OpenSSL-1.1
* VC_Redist.X-64.Exe

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs25.jpg)

Check your local modules folder to see if you already have this "Azure.Storage" and "AzureRM.Profile" modules installed (C:\Program Files\WindowsPowerShell\Modules)

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs9.jpg)
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs23.jpg)

If not, open an admin powershell window and run the following command to install them : 

~~~ powershell
install-module azure.storage
install-module azureRM.profile
~~~

OpenSSL can be downloaded [here](https://kb.firedaemon.com/support/solutions/articles/4000121705)
and the X64 VcRedist can be found [here](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)

Gather them all and group them in one subfolder like in the image above.

## Encryption Keys ##

The whole "why & how" is explained in [Part 2](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-2/). So I am just going to focus on functionality in this blog.  
We need to generate a Private and a public key for encryption purposes.  

Open a command prompt and navigate to 'C:\temp\sourcefiles\openssl\x64\bin'
(If the following command should fail, make sure you have the previously mentioned vc++ redist installed)

The following command will create our 2048 bits private key called "private_key.pem" :

~~~ shell
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
~~~

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs15.jpg)

Next, we need to generate the public key (public_key.pem) out of this private key :

~~~ shell
openssl rsa -pubout -in private_key.pem -out public_key.pem
~~~

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs16.jpg)

Alright, with both keys ready, we can move forward to the next steps.  

**Before you move forward, this next step is extremely important!**

Take your private key (private_key.pem) out of this folder and store it somewhere safe !! We will need it later on to decrypt our content, but we have to make sure it doesn't end up on our workstations that are enrolled via Autopilot.

## Creating the initial PSADT package ##

From an *admin* commandprompt, browse to C:\temp\psadtgui and launch the PSAPP_GUI.PS1

One the Application details tab, enter the following details :
* Application Vendor : OSCC (or your own company)
* Application Name : Copy logs to azure
* Application Version : 1.0
* RegID Year : 2008
* RegID Month : 03
* RegID Domain : be.oscc
* License Required : No
* Installation method : Script
* Script executable : FINDME (this will help us find the correct section later for further finetuning)
* Package binaries path : C:\temp\sourcefiles
* Destination package path (UNC) : \\localhost\c$\temp (or another share you have access to)

**Note :** If you want to have more details on the RegID or the fields in general, you can read up on it [here](https://github.com/TomDegreef/PSADT_GUI)

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs26.jpg)

If all details are completed, Click "Generate Application"

Once the tool has finished, you should have a subfolder called OSCC (or whatever you put in as vendor name) in C:\temp containing our initial PSADT package. We will now make the necessary changes in this package for everything to work.

Doublecheck the "files" subfolder in "C:\temp\OSCC\copy logs to azure_1.0_x86" to see if it has indeed the binaries we expect.

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs28.jpg)

## Making changes to our PSADT package ##

In "C:\temp\OSCC\copy logs to azure_1.0_x86" you will find a file called : Deploy-Application.PS1 . Edit it in whatever editor you prefer and search for the word "FINDME"

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs29.jpg)
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs30.jpg)

Remove that entire line "Execute-Process -Path 'FindME' " and replace it with the following script :

{% highlight powershell linenos %}
# Install azure storage powershell module
Copy-File -Path "$dirFiles\Azure.Storage\*" -Destination "C:\Program Files\WindowsPowerShell\Modules\Azure.Storage" -recurse

 # Install azure profile powershell module
Copy-File -Path "$dirFiles\AzureRM.profile\*" -Destination "C:\Program Files\WindowsPowerShell\Modules\AzureRM.profile" -recurse
        
# Install OpenSSL
       
Copy-File -Path "$dirFiles\openssl-1.1\*" -Destination "C:\openssl" -recurse

# Install VC++
Execute-Process -Path "$dirFiles\VC_redist.x64.exe" -Parameters '/install /quiet /norestart'

# Copy script locally
$ScriptLocation = "C:\LogsToAzure\Uploadlogstoazure_autopilot.ps1"
        
New-Folder -Path "C:\LogsToAzure"
Copy-File -Path "$dirsupportfiles\Uploadlogstoazure_autopilot.ps1" -Destination $ScriptLocation

$RunMyScript = "-executionpolicy bypass -Noninteractive -WindowStyle hidden -file $ScriptLocation"
#Create scheduled task
$trigger = New-ScheduledTaskTrigger -AtLogon
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument $RunMyScript 
$principal = New-ScheduledTaskPrincipal -UserId SYSTEM -LogonType ServiceAccount -RunLevel Highest
Register-ScheduledTask -TaskName "CopyLogs" -Description "CopyLogs" -Trigger $trigger -Action $action -Principal $principal

Get-ScheduledTask 
{% endhighlight %}

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs31.jpg)

Ok, so what have we done here ?
* Line 2 & 4 : Copy of the Azure Powershell modules to the correct subfolder
* Line 9 : Copy of the OpenSSL tool
* Line 12 : Installation of VC++
* Line 17 : Creation of the folder that we will use to gather the logs
* Line 18 : Copy of the script that will perform the log gathering
* remaining lines : creation of a scheduled task that will run at logon and trigger our script for the log gathering

Nearly There ! As you noticed on line 18, we copy a script to a folder. We still need to make this script.  

In the folder "C:\temp\OSCC\copy logs to azure_1.0_x86\SupportFiles" , create a new file called "Uploadlogstoazure_autopilot.ps1" (It has to match the script on line 18 of the previous script)

![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs32.jpg)

Edit this newly created file and put the following script as its content :

{% highlight powershell linenos %}
Import-Module azure.storage

$BlobProperties = @{

    StorageAccountName   = 'osdlogging'
    storSas              = '?sv=2019-12-12&st=2020-12-30T1<<censored>>'
    container            = 'osdlogs'
}

$TSHostname = $env:computername

$hostname = ($TSHostname).tolower()
$Timestamp = get-date -f yyyy-MM-dd-HH-mm-ss
$localpath = "C:\LogsToAzure\$hostname-$Timestamp"

New-Item -ItemType Directory -Path $localpath -Force
New-Item -ItemType Directory -Path $localpath\Panther -Force
New-Item -ItemType Directory -Path $localpath\Software -Force
New-Item -ItemType Directory -Path $localpath\Dism -Force
New-Item -ItemType Directory -Path $localpath\IME -Force

Get-ChildItem -Path C:\Windows\Panther | Copy-Item -Destination $localpath\Panther -Recurse
Get-ChildItem -Path C:\Windows\Logs\Software | Copy-Item -Destination $localpath\Software -Recurse
Get-ChildItem -Path C:\Windows\Logs\Dism | Copy-Item -Destination $localpath\Dism -Recurse
Get-ChildItem -Path 'C:\ProgramData\Microsoft\IntuneManagementExtension\Logs' | Copy-Item -Destination $localpath\IME -Recurse

write-host "compressing logfiles"
Compress-Archive -Path $localpath -DestinationPath "C:\LogsToAzure\$hostname-$Timestamp.zip"

write-host "Generating symmetric key"
C:\Openssl\x64\bin\openssl rand 100 > "C:\LogsToAzure\symmetric_keyfile.key"

write-host "Encrypt zipfile with symmetric key"
C:\Openssl\x64\bin\openssl enc -in "C:\LogsToAzure\$hostname-$Timestamp.zip" -out "c:\logstoazure\$hostname-$Timestamp.enc" -e -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -pass file:"C:\LogsToAzure\symmetric_keyfile.key"

write-host "Encrypt symmectric key with public key"
C:\Openssl\x64\bin\openssl rsautl -encrypt -inkey "C:\Openssl\x64\bin\public_key.pem" -pubin -in "C:\LogsToAzure\symmetric_keyfile.key" -out "C:\LogsToAzure\$hostname-$Timestamp.key.enc"

write-host "upload to azure"
$clientContext = New-AzureStorageContext -SasToken ($BlobProperties.storsas) -StorageAccountName ($blobproperties.StorageAccountName)

Set-AzureStorageBlobContent -Context $ClientContext -container ($BlobProperties.container) -File "c:\logstoazure\$hostname-$Timestamp.enc"
Set-AzureStorageBlobContent -Context $ClientContext -container ($BlobProperties.container) -File "C:\LogsToAzure\$hostname-$Timestamp.key.enc"

Unregister-ScheduledTask -TaskName "CopyLogs" -Confirm:$false
{% endhighlight %}

You could pretty much use the above script as-is. You just have to adjust the variables for the azure storage.

1) On line 5, set the StorageAccountName to the containername we created in the azure portal  
2) On line 6, replace the content of the storSAS variable with your "Write" SAS-key  
3) On line 7, provide the name of the blob-container we created in our preparation step using the storage explorer tool.

The full explanation of this script can be found in [Part 1](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-1/) and [Part 2](https://www.oscc.be/sccm/Logging-in-the-cloud-Part-2/). The changes are :

* Line 20 : creation of a folder for the Intune management extension logs
* Line 25 : Copy of the IME logs
* Line 45 : Removal of the scheduled task, so it runs just once !

## Validation ##

At this point, It probably makes sense to test the entire flow before we upload it to Intune.

Copy your full PSADT package to a test computer and from an *admin* command prompt, run the "Deploy-Application.exe".  
It shouldn't take long to run and the endresult should be :

* A folder called "C:\LogsToAzure" with a script in it called Uploadlogstoazure_autopilot.ps1
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs33.jpg)

* The azure Powershell modules in C:\Program Files\WindowsPowerShell\Modules
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs34.jpg)

* A scheduled task called "CopyLogs" in the root of your scheduled tasks library (launch task scheduler as admin if you don't see the task!)
* ![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs35.jpg)

If all those items are indeed where they are supposed to be, we can validate the next step and reboot this test-pc. Once you logon (doesn't need to be as an admin), your script should kick in action and do it's magic ;)

To validate this second piece, browse to the "C:\LogsToAzure" folder and you should see a bunch of files created 
![alt]({{ site.url }}{{ site.baseurl }}/images/azurelogs/azurelogs36.jpg)

Also, fire up your Azure storage explorer (see Part 1) and validate that indeed your logfiles are uploaded to the blob storage !

It makes sense to see if you can download & decrypt the files as well :) 

On the computer that you will use to download the logfiles, I expect that you have a working version of OpenSSL in C:\OpenSSL with the **private key (private_key.pem)** in the X64\Bin folder. (see beginning of this blog where we created those keys)

{% highlight powershell linenos %}
Import-Module azure.storage

$BlobProperties = @{

    StorageAccountName   = 'osdlogging'
    storSas              = '?sv=2019-12-12<<<rest of key is censored>>>'
    container            = 'osdlogs'
}
$clientContext = New-AzureStorageContext -SasToken ($BlobProperties.storsas) -StorageAccountName ($blobproperties.StorageAccountName)

$files = Get-AzureStorageBlob -container ($BlobProperties.container) -Context $clientContext 

foreach ($file in $files)
{
    write-host $file.name
    $last7chars = $file.name.SubString($($file.name).Length-7)
    write-host $last7chars
    if ($last7chars -eq "key.enc")
    {
        Write-host "Downloading the encryption key"
        Get-AzureStorageBlobContent -Destination "C:\LogsFromAzure" -container ($BlobProperties.container) -Context $clientContext -Blob $file.name
        $logfile = $file.name.SubString(0,$($file.name).Length-7) + "enc"
        Write-host "Downloading the matching encrypted logfiles"
        Get-AzureStorageBlobContent -Destination "C:\LogsFromAzure" -container ($BlobProperties.container) -Context $clientContext -Blob $logfile
        
        write-host "Decrypting key with private key"
        C:\Openssl\x64\bin\openssl rsautl -decrypt -inkey "C:\Openssl\x64\bin\private_key.pem" -in "C:\LogsFromAzure\$($file.name)" -out "C:\LogsFromAzure\symmetric_keyfile_dec.key"

        write-host "decrypting logfiles"
        C:\Openssl\x64\bin\openssl enc -in "C:\LogsFromAzure\$logfile" -out "C:\LogsFromAzure\decrypted.zip" -d -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -pass file:"C:\LogsFromAzure\symmetric_keyfile_dec.key"

        write-host "expanding logfiles"
        Expand-Archive -Path "C:\LogsFromAzure\decrypted.zip" -DestinationPath "C:\LogsFromAzure"

        write-host "cleaning up"
        Remove-Item -Path "C:\LogsFromAzure\decrypted.zip" -Force
        Remove-item -Path "C:\LogsFromAzure\symmetric_keyfile_dec.key" -force
        Remove-Item -Path "C:\LogsFromAzure\$logfile" -force
        Remove-Item -Path "C:\LogsFromAzure\$($file.name)" -Force

        Remove-AzureStorageBlob -container ($BlobProperties.container) -Context $clientContext -Blob $logfile
        Remove-AzureStorageBlob -container ($BlobProperties.container) -Context $clientContext -Blob $file.name
  
    }

}
{% endhighlight %}

Again, don't forget to adjust the variables for the azure storage on lines 5,6,7 !!

Running this script should retrieve the logs and decrypt them.

Fingers crossed that everything worked as it should! If not, make sure to revisit all steps and execute the scripts in small pieces to see what could have gone wrong.  
If everything worked as expected, we can do the final integration into Intune & Autopilot!

## Creation of the Intune Package ##
