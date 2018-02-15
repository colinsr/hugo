---
title: "Azure Application Gatway - Part 2"
date: 2018-01-29T10:43:42-05:00
tags: ["load balancing", "azure"]
categories: ["cloud", "azure"]
---

This is a continuation from part one on setting up an [Azure Application Gateway Part 1](/post/azureapplicationgatwaypart1). 

Next on our hit list is creating an Azure VNet to hold our AppGW.  The AppGW has to live in an AppGW subnet, the only other thing that can reside in the same subnet are other AppGW's and hopefully you'll only need one since the AppGW is capable of serving up to 20 sites, with various limits around authentication certs for backend re-encryption.
``` powershell
$gwSubnetName = 'appgwsubnet'
$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name $gwSubnetName `
    -AddressPrefix 10.0.0.0/28
$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-RG `
    -Location $location -AddressPrefix 10.0.0.0/16 -Subnet $subnet
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name $gwSubnetName `
    -VirtualNetwork $vnet
```

OK, now we can get down to the business of building up our AppGW config. There are a few moving parts involved so let's take a minute to review what they are.

The first concept we'll talk about is the Backend Pool.  This is essentially the destination that you'd like to route the traffic to.  You can use either IP or FQDN to route the traffic.  These could be VMs, Web Apps a Load Balancer - Backend Pool targets are pretty flexible.

Next up are the HTTP settings.  This is where you configure the Backend Pool's destination port, cookie based session affinity, and if you're using end to end SSL you'll need to add the add the public key of the Backend Pool in .CER format.  It's pretty straightforward to extract the .CER from a PFX.  Note, you can also attach a custom probe, which you can use to verify the health of Backend Pools that are not listening on standard ports.  I bumped into an issue where I configured a custom probe to check the health of Backend Pools which were listening on port 443 and my Backend Pools were showing a status of 'Unknown' - do yourself a favor and use the default probes if you want to keep your hair.  There is also another scenario where you'll need to use a custom probe even when using default ports - when you want to pull the hostname from the Backend Http Settings.

That brings us to the Frontend port configuration.  When we create our AppGW we'll need to create a public IP that we'll attach to the AppGW.  When we have an IP we can go about creating some frontend ports.  Traffic will hit the frontend port and then based on some listeners get routed to the appropriate Backend Pool.  We're going to configure two ports in our demo - 80 and 443.  We'll setup a redirect configuration to send traffic coming in over http on port 80 to https on 443.

I've just mentioned listeners, so now we can cover what part a listener plays. The listener has a few properties, the frontend IP, the frontend port, the protocol (HTTP/HTTPS), and if HTTPS is selected we'll associate the SSL cert as well.

And finally, the rule.  The rule is the glue that holds the listener with the backend pool.  Rule's have a reference to the listener, Backend Pool and backend HTTP Setting.  Hopefully this will all make a bit more sense once we cover the "how to setup" bits.

Now that we've gotten some of the terminology out of the way we can start building this thing up.  We need to create a public IP address to assign to our AppGW - bear in mind this is a dynamic IP, static IP's are not supported currently on the AppGW.  I've been told by Azure support that the IP _should_ only change if you stop the Application Gateway... 
This IP will not be used in your DNS mapping as it's dynamic and it would suck to have all your sites die when Azure decides to reassign you a new IP.  Instead we'll want to set up a CNAME entry that points from your custom domain name to directly to your AppGW's FQDN - we'll get to that step in a bit.
``` powershell
$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -name AppGWIP `
                -location $location -AllocationMethod Dynamic 
$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet
```
Next up are the frontend ports.
``` powershell
$appGWFEIpConfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name "GatewayFrontEndIp" `
                    -PublicIPAddress $appGatewayIp
$httpsFrontEndPort = New-AzureRmApplicationGatewayFrontendPort -Name "https" -Port 443
$httpFrontEndPort = New-AzureRmApplicationGatewayFrontendPort -Name "http" -Port 80  
```

Remember when I mentioned the custom probes on default ports?  Well now we can get into the deets of that.
We need to set up probe health response match, which we'll set to a response in the 200-399 range.  Then we'll create a probe config for each one of our Backend Pools.  
``` powershell
$match = New-AzureRmApplicationGatewayProbeHealthResponseMatch `
            -StatusCode 200-399

# Get the new probes
$site1Probe  = New-AzureRmApplicationGatewayProbeConfig  -Name "site1Probe" -Protocol Https `
        -Path / -Interval 30 -Timeout 120 -UnhealthyThreshold 3 -PickHostNameFromBackendHttpSettings -Match $match

$site2Probe  = New-AzureRmApplicationGatewayProbeConfig  -Name "site1Probe" -Protocol Https `
        -Path / -Interval 30 -Timeout 120 -UnhealthyThreshold 3 -PickHostNameFromBackendHttpSettings -Match $match
```

Moving on now to setting up the Backend Http Settings for our two Backend Pools.
``` powershell
$site1HttpSettings = New-AzureRmApplicationGatewayBackendHttpSettings -Name "site1HttpSettings" `
        -Port 443 -Protocol Https -CookieBasedAffinity Disabled -RequestTimeout 120 `
        -HostName $customHostName1 -Probe $site1Probe

$site2HttpSettings = New-AzureRmApplicationGatewayBackendHttpSettings -Name "site2HttpSettings" `
        -Port 443 -Protocol Https -CookieBasedAffinity Disabled -RequestTimeout 120 `
        -HostName $customHostName2 -Probe $site2Probe
```

Now we can work on setting up the Backend Pools for our two Web Apps.
``` powershell
$pool1 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendFqdns $defaultHostName1
$pool2 = New-AzureRmApplicationGatewayBackendAddressPool -Name pool02 -BackendFqdns $defaultHostName2
```

On to the Listeners, for the HTTPS listeners we need to publish our cert up to the Application Gateway so we'll get that setup and then create our HTTPS and HTTP listeners.
``` powershell
$gwCert = New-AzureRmApplicationGatewaySSLCertificate -Name ohioazurewildcard `
    -CertificateFile $pfxPath -Password $passString

# HTTPS listeners
$site1Listener443 = New-AzureRmApplicationGatewayHttpListener `
    -Name "Site1Listener443" `
    -Protocol Https `
    -FrontendIPConfiguration $appGWFEIpConfig `
    -FrontendPort $httpsFrontEndPort `
    -HostName $customHostName1 `
    -RequireServerNameIndication true  `
    -SslCertificate $gwCert

$site2Listener443 = New-AzureRmApplicationGatewayHttpListener `
        -Name "Site2Listener443" `
        -Protocol Https `
        -FrontendIPConfiguration $appGWFEIpConfig `
        -FrontendPort $httpsFrontEndPort `
        -HostName $customHostName2 `
        -RequireServerNameIndication true  `
        -SslCertificate $gwCert

# HTTP listeners
$site1Listener80 = New-AzureRmApplicationGatewayHttpListener `
        -Name "Site1Listener80" `
        -Protocol Http `
        -FrontendIPConfiguration $appGWFEIpConfig `
        -FrontendPort $httpFrontEndPort `
        -HostName $customHostName1

$site2Listener80 = New-AzureRmApplicationGatewayHttpListener `
        -Name "Site2Listener80" `
        -Protocol Http `
        -FrontendIPConfiguration $appGWFEIpConfig `
        -FrontendPort $httpFrontEndPort `
        -HostName $customHostName2 
```

Now that we have our listeners created we can go about setting up the 80 => 443 site redirects.  You can see here we're creating 2 redirection configurations - one for each site.  The important piece here is the `-TargetListener`.
``` powershell
$site1RedirectConfig = New-AzureRmApplicationGatewayRedirectConfiguration -Name "redirectSite1" `
    -RedirectType Permanent -TargetListener $site1Listener443

$site2RedirectConfig = New-AzureRmApplicationGatewayRedirectConfiguration -Name "redirectSite2" `
    -RedirectType Permanent -TargetListener $site2Listener443
```

We are getting close to finished here.  We still need to setup the routing rules.  We'll need 2 routing rules for each site.  One will be the redirect routing rule, and the other will route the traffic to the backend pool specific to the hostname in the listener.
``` powershell 
# Redirect routing rules
$site1RedirectRule = New-AzureRmApplicationGatewayRequestRoutingRule -Name "site1RedirectRule" `
    -RuleType Basic -HttpListener $site1Listener80 -RedirectConfiguration $site1RedirectConfig
$site2RedirectRule = New-AzureRmApplicationGatewayRequestRoutingRule -Name "site2RedirectRule" `
    -RuleType Basic -HttpListener $site2Listener80 -RedirectConfiguration $site2RedirectConfig
# Backend pool routing rule
$site1Rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name "site1Rule" `
    -RuleType Basic -HttpListener $site1Listener443 -BackendHttpSettings $site1HttpSettings `
    -BackendAddressPool $pool1

$site2Rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name "site2Rule" `
    -RuleType Basic -HttpListener $site2Listener443 -BackendHttpSettings $site2HttpSettings `
    -BackendAddressPool $pool2
```

Next we can set the sku for the Application Gateway.  Remember WAF is an extra charge so you'll want to make sure you choose the appropriate sku here.  Also, Azure recommends 2 Application Gateways for high availability.  Also, make sure to create a WAF configuration so that you can enable your WAF when the Application Gateway comes up.
``` powershell
$Sku = New-AzureRmApplicationGatewaySku -Name "WAF_Medium" -Tier "WAF" -Capacity 2

$wafConfig = New-AzureRmApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode "Prevention"
```

And here we are, finally about to create our Application Gateway.  We need to make sure and pass along all the required configuration objects.
``` powershell
$appgw = New-AzureRmApplicationGateway -Name appgw `
            -ResourceGroupName $rgName `
            -Location $location `
            -BackendAddressPools $pool1,$pool2 `
            -BackendHttpSettingsCollection $site1HttpSettings,$site2HttpSettings `
            -FrontendIpConfigurations $appGWFEIpConfig `
            -GatewayIpConfigurations $gipconfig  `
            -FrontendPorts $httpsFrontEndPort,$httpFrontEndPort `
            -HttpListeners $site1Listener443,$site2Listener443,$site1Listener80,$site2Listener80 `
            -RequestRoutingRules $site1RedirectRule,$site1RedirectRule,$site1Rule,$site1Rule `
            -Sku $Sku `
            -WebApplicationFirewallConfig $wafConfig
            -SslCertificates $gwCert `
            -Probes $site1Probe,$site2Probe `
            -RedirectConfigurations $site1RedirectConfig,$site2RedirectConfig
```

These Application Gateways take a little while to deploy, so if you've made it this far now is the perfect time to go find yourself a fresh cup 'o joe.  When the deployment is complete, we'll need to update the CNAME records that we previously pointed directly at the Web Apps to point at the Application Gateway.

If your DNS zone is hosted in Azure then you're a few lines of Powershell away from a wrap.  First we'll get the FQDN of the new Application Gateway, then we update the entries.
``` powershell
$gatewayFQDN = ((Get-AzureRmPublicIpAddress -ResourceGroupName $rgName -Name AppGWIP).DnsSettings).Fqdn

$site1RecordSet = Get-AzureRmDnsRecordSet -Name "site1" -RecordType CNAME -ZoneName "OhioAzure.com" -ResourceGroupName OhioAzureDomain
$site1RecordSet.Records[0].Cname = $gatewayFQDN
Set-AzureRmDnsRecordSet -RecordSet $site1RecordSet

$site2RecordSet = Get-AzureRmDnsRecordSet -Name "site2" -RecordType CNAME -ZoneName "OhioAzure.com" -ResourceGroupName OhioAzureDomain
$site2RecordSet.Records[0].Cname = $gatewayFQDN
Set-AzureRmDnsRecordSet -RecordSet $site2RecordSet
```

So there you have it.  A working end to end SSL multi-site redirect enabled WAF Application Gateway.  Enjoy.

If you're using a self signed cert like I did then your browser will give you a warning - feel free to ignore.