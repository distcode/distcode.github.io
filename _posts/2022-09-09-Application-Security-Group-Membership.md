---
layout: post
title: ASG Membership 
tags: [about]
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

```powershell
Get-AzNetworkInterface -ResourceGroupName RG-Az040 -Name NIC-VM1
| Select-Object -ExpandProperty IpConfigurations
| Select-Object -ExpandProperty ApplicationSecurityGroups
| Select-Object -Property Id

# or short:

(Get-AzNetworkInterface -ResourceGroupName RG-Az040 -Name NIC-VM1).IpConfigurations.ApplicationSecurityGroups.Id
```


<!--
### In which ASGs is an Azure VM a member?

This articles shows you a solution to check in which Application Secrity Groups an Azure VM is a member. I'm using a PowerShell script which could you convert it into a function.
-->



