---
title: "Azure Application Gateway - Part 1"
date: 2018-01-29T10:43:38-05:00
tags: ["load balancing", "azure"]
categories: ["cloud", "azure"]
---

This is going to be a chronicling of how to setup an Azure Application Gateway using a custom wildcard certificate 
(You should be thinking "*.foobar.com").

## The Background
I've gone through some basic tutorials with the Azure Application Gateway (AppGW) in the past, but recently I encountered a scenario where a client needed me to setup an AppGW that could host 4 Azure Web Apps running an an Azure App Service Environment.
Some of the Application Gateway settings are not available in the portal, so we'll use Powershell for this particular implementation - let's get started with the configuration required to get this AppGW working properly.
### App GW Basics
In this tutorial I'll walk through the process of deploying an Azure Resource Group, with two Azure Web Apps running on custom subdomains, both using a wild card SSL Certificate and an Azure Application Gateway to be our security guard/traffic director.
You can use this pattern to host up to 20 websites hosted in Azure Web Apps, to host them on IaaS there are some tweaks that will need to happen.

We'll break this up into two parts to break up some of the monotony of the configuration.  Part one will be getting our self signed wildcard certificate, VNET and Azure Web Apps configiured, then in part two we'll go over the AppGW configuration in detail.

First things first, we need to login to our Azure subscription.
If you haven't already setup a profile for your Azure account, now would be a good time to do so to avoid the process of logging in manually every time you want to use Azure Powershell.
``` powershell
Login-AzureRmAccount
Save-AzureRmContext -Path C:\MyProfile.json
```

With that out of the way, in the future all you have to do to login is import your Azure context like so:
``` powershell
Import-AzureRmContext -Path 'C:\MyProfile.json' | Out-Null
```

And now select the target Subscription.
``` powershell
Get-AzureRmSubscription | 
    Select Name,SubscriptionId | 
    Out-Gridview -Title "Choose your Azure Subscription" -OutputMode Single | 
    Select-AzureRmSubscription
```

Now that we've selected the target Azure subscription we can get down to business.
We can set the Azure region we'd like to deploy our resources to, and create the resource group to hold all our stuff.

``` powershell
$location = 'West US'
$rgName = 'appgateway-rg'
$resourceGroup = New-AzureRmResourceGroup -Name $rgName -Location $location
```
Let's get the SSL certificate out of the way now.  You have two options here, self-signed and signed.
For demo purposes we'll go with the free self signed wildcard cert - in a production environment you'll definitely want to us a signed cert.
``` powershell
# Use signed certificate like so:
# $pfxPath= 'C:\...\foobarWildCard.pfx'
# $pfxPassString = 'P@ssw0rd5!'
# $pfxPassSecure = ConvertTo-SecureString -String $pfxPassString -Force -AsPlainText

# Or create a self signed cert for testing
$cert = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname *.ohioazure.com
$passString = 'P@ssw0rd5!'
$passSecure = ConvertTo-SecureString -String $passString -Force -AsPlainText
$pfxPath = 'c:\wildcard.pfx'
$path = 'cert:\localMachine\my\' + $cert.thumbprint 
Export-PfxCertificate -cert $path -FilePath $pfxPath -Password $passSecure
```

Next up, we'll create an Azure App Service Plan to run our Web Apps.  Make sure to set the Tier at least Shared so that we have access to Custom Domains. 
``` powershell
$appServicePlanName = 'ohioazure'

New-AzureRmAppServicePlan -Name $appServicePlanName -Location $location `
    -ResourceGroupName $rgname -Tier Standard
```

Alright, we're ready now to create our first Azure Web app.
``` powershell
$customHostName1 = 'site1.ohioazure.com'
$webAppName1 = 'site1-ohio'
$defaultHostName1 = "$webAppName1.azurewebsites.net"

New-AzureRmWebApp -Name $webappname1 -Location $location `
     -AppServicePlan $appServicePlanName -ResourceGroupName $rgName
```

I think it's pretty painless to create a Web App inside - but now let's have a little fun and start customizing it.  We'll want to bind this Web App to our custom hostname and in order to do so we need to add a CNAME record to our DNS zone.  Fortunately, my DNS zone is hosted in Azure so it's a straight forward process.  See the docs for your DNS provider on exactly how to add a CNAME record for your Web App.
After that we'll add the SSL binding with our newly created self signed wildcard certificate.
``` powershell
#TODO do not proceed until CNAME record is mapped to the Web App's FQDN
New-AzureRmDnsRecordSet -Name "site1" -RecordType CNAME -ZoneName "OhioAzure.com" `
    -ResourceGroupName "OhioAzureDomain" -Ttl 300 `
    -DnsRecords (New-AzureRmDnsRecordConfig -CNAME $defaultHostName1)

Set-AzureRmWebApp -Name $webAppName1 -ResourceGroupName $rgName `
    -HostNames $customHostName1,$defaultHostName1
New-AzureRmWebAppSSLBinding -WebAppName $webappname1 -ResourceGroupName $rgName -Name $customHostName1 `
    -CertificateFilePath $pfxPath -CertificatePassword $passString -SslState SniEnabled
```

Repeat for your second (and any other subsequent Web Apps you may be runing)...
``` powershell
$customHostName2 = 'site2.ohioazure.com'
$webAppName2 = 'site2-ohio'
$defaultHostName2 = "$webAppName2.azurewebsites.net"

New-AzureRmWebApp -Name $webappname2 -Location $location `
     -AppServicePlan $appServicePlanName -ResourceGroupName $rgName

New-AzureRmDnsRecordSet -Name "site2" -RecordType CNAME -ZoneName "OhioAzure.com" `
    -ResourceGroupName "OhioAzureDomain" -Ttl 300 `
    -DnsRecords (New-AzureRmDnsRecordConfig -Cname $defaultHostName2)

Set-AzureRmWebApp -Name $webAppName2 -ResourceGroupName $rgName `
    -HostNames $customHostName2,$defaultHostName2

New-AzureRmWebAppSSLBinding -WebAppName $webappname2 -ResourceGroupName $rgName -Name $customHostName2 `
    -CertificateFilePath $pfxPath -CertificatePassword $passString -SslState SniEnabled
```

I think now would be a good time to set the default document for our Web Apps so that we know which one we're hitting.  There are many different ways to go about this step but for ease of use we'll use FTP.
``` powershell
$publishProfile1 = [XML](Get-AzureRmWebAppPublishingProfile -Name $webappname2 `
    -ResourceGroupName $rgName `
    -OutputFile null)
$username1 = $publishProfile1.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userName").Value
$password1 = $publishProfile1.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userPWD").Value
$ftpUrlSite1 = $publishProfile1.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").Value

$file1 = "c:\temp\site1.html"
$webAppName1 | Out-File $file1

$webclient = New-Object -TypeName System.Net.WebClient
$webclient.Credentials = New-Object System.Net.NetworkCredential($username1,$password1)
$uri1 = New-Object System.Uri("$ftpUrlSite1/index.html")
$webclient.UploadFile($uri1, $file1)
$webclient.Dispose()

# In our example repeat this process for $webAppName2
```

While that method is effective at what we're trying to solve, the code needs to be repeated N times depending on how many Web Apps you're going to deploy.  Do yourself a favor and pull that code out into a function and pass in the moving targets as params - again since this is a demo I'm going with quick and extremely dirty here.

Urghh, after reading through my script again I had no choice but to slap this in here.  If you go this route add this function to the top of your script and replace the duplicated code with a call to this function.
``` powershell
function Set-IndexOnWebApp {
    Param(
        [string] $webAppName,
        [string] $resourceGroupName,
        [string] $filePath
    )
    $pubProfile = [XML](Get-AzureRmWebAppPublishingProfile -Name $webAppName `
        -ResourceGroupName $resourceGroupName `
        -OutputFile null)
    $user = $pubProfile.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userName").Value
    $pass = $pubProfile.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@userPWD").Value
    $ftpUrl = $pubProfile.SelectNodes("//publishProfile[@publishMethod=`"FTP`"]/@publishUrl").Value

    $webAppName | Out-File $filePath

    $client = New-Object -TypeName System.Net.WebClient
    $client.Credentials = New-Object System.Net.NetworkCredential($user,$pass)
    $uri = New-Object System.Uri("$ftpUrl/index.html")
    $client.UploadFile($uri, $filePath)
    $client.Dispose()
}
```

Once that's loaded up you can update the landing page for the site like so:
``` powershell
Set-IndexOnWebApp -webAppName $webAppName1 `
                  -resourceGroupName $rgName `
                  -filePath $file1

Set-IndexOnWebApp -webAppName $webAppName2 `
                  -resourceGroupName $rgName `
                  -filePath $file2
```

That pretty much wraps up the Web App configuration required to get our AppGW working.  We'll pick up in part two of this series getting our AppGW deployed out to Azure.

To continue this tutorial and pick up where we left off go to [Azure Application Gateway Part 2](/post/azureapplicationgatwaypart2).