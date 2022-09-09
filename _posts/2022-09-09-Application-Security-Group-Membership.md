---
layout: post
title: ASG Membership 
tags: [Azure, IaaS, PowerShell]
hide_title: true 
excerpt_separator: <!--more-->
---

# Application Security Group Members

This post describes how to get all members of an Application Security Group in Azure. I'm using a PowerShell script which could you convert into a function.
<!--more-->

An Application Security Group (ASG) in Azure offers you to group Azure VMs. This groups will be used in Network Security Groups Rules to allow or deny network traffic between that groups. For more information see the Microsoft documentation of [ASG](https://docs.microsoft.com/en-us/azure/virtual-network/application-security-groups) and [NSG](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview).

Unfortunately there is no way to get the member of an ASG in the Azure Portal. But with PowerShell you could get that information.

First, the information of an ASG membership is stored in the ipconfigation of VM's network interface. Use the following command to see the ASG Id of an ipconfiguration of a NIC with the name NIC-VM1:

{% highlight powershell linenos %}
Get-AzNetworkInterface -ResourceGroupName RG-Az040 -Name NIC-VM1
| Select-Object -ExpandProperty IpConfigurations
| Select-Object -ExpandProperty ApplicationSecurityGroups
| Select-Object -Property Id

# or short:

(Get-AzNetworkInterface -ResourceGroupName RG-Az040 -Name NIC-VM1).IpConfigurations.ApplicationSecurityGroups.Id
{% endhighlight %}

The result shows you an Azure resource id.

Even though there is a cmdlet to get information of an ASG, it does not show's the members. The cmdlet is `Get-AzApplicationSecurityGroup`and the result could be as follows:

{% highlight powershell linenos %}
ProvisioningState : Succeeded
ResourceGroupName : RG-Az040
Location          : westeurope
ResourceGuid      : 
Type              : Microsoft.Network/applicationSecurityGroups
Tag               : {}
TagsTable         : 
Name              : ASG-FrontEnd
Etag              : W/"27b2c5b0-277e-4ca4-8059-e8fc230ddc01"
Id                : /subscriptions/xxxxxxxx-4d2c-47f6-953c-a9dc64006f0e/resourceGroups/RG-Az040/providers/Microsoft.Network/applicationSecurityGroups/ASG-FrontEnd
{% endhighlight %}

So, now I could start creating the script. First, I create a param() section to be able to pass the name of an ASG.

{% highlight powershell linenos %}
[cmdletbinding()]
param (
    [string]$ASGName
    )
{% endhighlight %}

Then I check if the given group realy exists:

{% highlight spowershell linenos %}
$ErrorActionPreference = 'Stop'

# get ASG details
try {
    $asg = Get-AzResource -Name $ASGName
}
catch {
    $error[0] | Format-List * -Force
    return
}

if ( $null -eq $asg)
{
    "No ASG with the name $ASGName found."
    return
}
{% endhighlight %}

Now, the ASG is saved in the variable $asg. For the remaining script, the resource location is important, because only VM in the same location is able to be a member of the ASG. Therefore the ASG location is saved in the variable $asgregion.

{% highlight spowershell linenos %}
$asgregion = $asg.Location
{% endhighlight %}

Next, the scripts searches for all VMs in that location and for all VMs the NIC configuration is checked.

{% highlight spowershell linenos %}
$vms = Get-AzVM -Location $asgregion
foreach ( $vm in $vms )
{
    foreach ( $nic in $vm.NetworkProfile.NetworkInterfaces)
    {
        ...        
    }
}
{% endhighlight %}

Inside the second foreach statement, all ipconfigs are searched and checked for ASG ids.

{% highlight spowershell linenos %}
$nicdetail = Get-AzNetworkInterface -ResourceId $nic.Id
    foreach ( $ipconfig in $nicdetail.IpConfigurations )
    {
        if ( $ipconfig.ApplicationSecurityGroups.Count -ge 1 )
        { 
            foreach ( $asg in $ipconfig.ApplicationSecurityGroups)
            {
                ...
            }
        }
    }
{% endhighlight %}

Only if an ASG configuration is found, the next inner if will be executed to see if the ASG name of the parameter is the same as for that ipconfiguration. To extract the name of the ASG of the id the split operator is used and then the last value in the resulting array `$array[-1]` contains the name.

{% highlight spowershell linenos %}
if ( $asg.id.split('/')[-1] -eq $ASGName )
{
    ...
}
{% endhighlight %}

Finally, the script collects some data of the VM and NIC an returs all that in as an array. The variable `$foundVMS` was created earlier.

{% highlight spowershell linenos %}
$foundVM = [foundASGMember]::new()
$foundVM.VM = $vm
$foundVM.NIC = $nicdetail
$foundVM.NICIPConfiguration = $ipconfig
$foundVM.NICIPConfigName = $ipconfig.Name
$foundVM.VMName = $vm.Name
$foundVM.NICName = $nic.Id.split('/')[-1]
$foundVMs += $foundVM
{% endhighlight %}

The whole script could be copied from here:

{% highlight spowershell linenos %}
#Requires -Modules Az.Compute, Az.Network

[cmdletbinding()]
param (
    [string]$ASGName = 'ASG-FrontEnd'
)
    
class foundASGMember {
    [string]$VMName
    [string]$NICName
    [string]$NICIPConfigName
    [Microsoft.Azure.Commands.Compute.Models.PSVirtualMachine]$VM
    [Microsoft.Azure.Commands.Network.Models.PSNetworkInterface]$NIC
    [Microsoft.Azure.Commands.Network.Models.PSNetworkInterfaceIPConfiguration]$NICIPConfiguration
}
    
$ErrorActionPreference = 'Stop'

# get ASG details
try {
    $asg = Get-AzResource -Name $ASGName
}
catch {
    $error[0] | Format-List * -Force
    return
}

if ( $null -eq $asg) {
    "No ASG with the name $ASGName found."
    return
}
$asgregion = $asg.Location
# $asgrg = $asg.ResourceGroupName

$foundVMs = @()

# Check VMs in same region
$vms = Get-AzVM -Location $asgregion
foreach ( $vm in $vms ) {
    foreach ( $nic in $vm.NetworkProfile.NetworkInterfaces) {
        $nicdetail = Get-AzNetworkInterface -ResourceId $nic.Id
        foreach ( $ipconfig in $nicdetail.IpConfigurations ) {
            if ( $ipconfig.ApplicationSecurityGroups.Count -ge 1 ) {
                foreach ( $asg in $ipconfig.ApplicationSecurityGroups)
                { 
                    if ( $asg.id.split('/')[-1] -eq $ASGName ) {
                        $foundVM = [foundASGMember]::new()
                        $foundVM.VM = $vm
                        $foundVM.NIC = $nicdetail
                        $foundVM.NICIPConfiguration = $ipconfig
                        $foundVM.NICIPConfigName = $ipconfig.Name
                        $foundVM.VMName = $vm.Name
                        $foundVM.NICName = $nic.Id.split('/')[-1]
                        $foundVMs += $foundVM
                    }
                }
            }
        }
        
    }
}
return $foundVMs
{% endhighlight %}

Name the script *Get-AzASGMember* and use it in the following ways (examples):

{% highlight spowershell linenos %}
.\Get-AsASGMember.ps1 -ASGName ASG-FrontEnd

$result = .\Get-AsASGMember.ps1 -ASGName ASG-FrontEnd
$result.VM
{% endhighlight %}

Feel free to extend and improve the script to meet your personal requirements.