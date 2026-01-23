+++
date = '2026-01-07T09:57:52-08:00'
title = 'Active Directory Lab'

image = "/post/images/active_directory.jpg"
tags = ["active-directory", "windows", "Security"]
categories = ["Security"]
+++

# Active Directory

## Wikipedia
> Active Directory (AD) is a directory service developed by Microsoft for Windows domain networks. Windows Server operating systems include it as a set of processes and services.<sup>[1][2]</sup> Originally, only centralized domain management used Active Directory. However, it ultimately became an umbrella title for various directory-based identity-related services.<sup>[3]</sup>

## ChatGPT
> Active Directory (AD) is a centralized directory and identity management system used in Windows-based networks. It allows organizations to manage users, computers, permissions, and security policies from one place.

## What does this mean?
Active Directory is a term that I have encountered a few times. But every time it is accompanied by a vague and wide sweeping definition like the ones seen above leaving me with an arms length understanding. So today I am going to attempt to squash my ignorance and implement a very VERY basic AD network. The plan is to create a a few virtual machines to simulate a basic network with AD governing it. 

## AD Basics
Before I begin I want to familiarize myself with some AD terminology. 
- **Domain Controller (DC)**: The server that is running AD and holds the directory database. 
- **Domain**: A logical boundary that contains objects such as users, computers, and groups, and provides centralized authentication and authorization.
- **Tree**: A collection of domains with contiguous namespace (sales.corp.com & engineering.corp.com).
    - Namespace refers to the "corp" in that domain. Both sales and engineering both share the same namespace
- **Forest**: A collection of one or more trees
- **Organization Unit (OU)**: A container within a domain that organizes users computers or other OUs. 

## 1. Set up the VMs
I will begin with only two VMs. 
1. **Windows server 2022**: This will be my DC.
2. **Windows 11**: This will be the workstation. 

VirtualBox by Oracle with be my hypervisor of choice. Setting these up is pretty straight forward. Just downloading the ISOs from the Microsoft website and configuring the system requirements for each system. 

One small "gotcha" here is ensuring that both VMs are on the same virtual network. Under the "network" tab of each VM you can configure the network setting for each system. The default is "NAT". NAT lets your VM access external networks (like the internet) by sharing the host machine’s IP address. This will not work for us since other machines cannot initiate connection to the VM. We will need to select "Internal Network". This creates a private virtual switch inside VirtualBox. This means our devices cannot connect to the outside internet but they can connect to each other. 

## 2. Windows Server Config
We begin with the basic set up, language, region, installation type, etc. The key here is the administrator password. This will function as our master password and allow us access into the DC to configure everything else. It has access to just about everything so I kept it very secure by setting it as "Adminpass123". Upon reboot we can now login as the administrator. For simplicity sake I renamed this machine to DC01. 

Now it is time to configure the network. The first thing we will do is set up a static IP. But first we must get out network adapter which, in powershell can be done with the following:
```powershell
Get-NetAdapter
```
Taking note of the name of this adapter, we can now reconfigure. We can rename it to a name of our choosing and also set up our IP address and our subnet mask. In this case I'm using ```192.168.50.10``` and a subnet mask of the first 24 bits. In laymans terms, this just means all devices on this subnet can only change values after the last ".". 
```powershell
New-NetIPAddress `
  -InterfaceAlias "Ethernet0" `
  -IPAddress 192.168.50.10 `
  -PrefixLength 24
```

Now we will configure DNS. This is critical for AD. This DC will act as the DNS server for the rest of the network. If we use a basic DNS server like Google's ```8.8.8.8``` it will have no way of finding our other devices. Fortunately, this is much easier than I originally thought. All it takes is the following:
```powershell
Set-DnsClientServerAddress `
  -InterfaceAlias "Ethernet0" `
  -ServerAddresses 192.168.50.10
```

Now we must turn this server into the DC. We still don't have Active Directory Domain Services installed. So we will do that with the following:
```powershell
Install-WindowsFeature AD-Domain-Services,DNS -IncludeManagementTools
```

Now we promote our machine to DC. We will do this by creating a new [forest](##AD-Basics). Within our forest, our current machine is the DC and the DNS server. Once again, we are forced to reboot. After rebooting we can login back in as the admin and double check everything is working with the following:
```powershell
Get-Service ADWS
Get-Service DNS
Get-ADDomain
```

## 3. Basic AD Config
Now we will create some basic [organization units](##AD-Basics). One for workstations, one for users, and one for groups. That can be done with the following:
```powershell
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Users" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Groups" -Path "DC=lab,DC=local"
```

We can verifying their creation with the following:
```powershell
Get-ADOrganizationalUnit -Filter *
```

We can now create our first users, lets call her Alice:
```powershell
New-ADUser `
  -Name "Alice Johnson" `
  -SamAccountName ajohnson `
  -UserPrincipalName ajohnson@lab.local `
  -Path "OU=Users,DC=lab,DC=local" `
  -AccountPassword (Read-Host -AsSecureString "Password") `
  -Enabled $true
```

We can create a group as well:
```powershell
New-ADGroup `
  -Name "IT-Admins" `
  -GroupScope Global `
  -GroupCategory Security `
  -Path "OU=Groups,DC=lab,DC=local"
```

Then we can add lovely Alice to our newly created group:
```powershell
Add-ADGroupMember -Identity "IT-Admins" -Members ajohnson
```

This create simplicity when applying security rules. If we were to get a new IT admin, we wouldn't need to find all of Alice's permission and copy them over to this new user we could simply add them to the already created IT-Admins group and this would be automatically done. This way we don't have to mange users, we can just worry about groups. Similar to the principles of [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control). 

## 4. Configure the Client
At this point, Active Directory is installed, DNS is configured, and basic directory objects exist. The final step is to introduce a client machine and allow it to authenticate against the domain. Computer Objects in Active Directory Unlike users and groups, computer objects do not need to be created manually in most environments. When a Windows machine joins a domain, Active Directory automatically creates a corresponding computer object and establishes a trust relationship between the machine and the domain controller. By default, this computer object is placed in the built-in Computers container. But we will move this into a dedicated Organizational Unit so that policies and configuration can be applied consistently.

Before joining the domain, the workstation must be able to locate the domain controller. Because Active Directory relies heavily on DNS, the workstation’s DNS server must point to the domain controller, not an external resolver. In the Windows setting we can configure a custom IP address and point to a custom DNS server using these settings:

```
IP Address: 192.168.50.20
Subnet Mask: 255.255.255.0
DNS Server: 192.168.50.10
```

We can test connectivity of the devices using ping:
```
ping 192.168.50.10
ping dc01
ping dc01.lab.local
```

We could now join the domain. In Windows setting we can select out domain as ``lab.local```. We then authenticate with admin credentials and reboot. 


## 5. Move Workstation to Workstations OU
To move this devices from the default ```computers``` container over to the ```workstations``` OU we first find the name of the computer with:
```powershell
Get-ADComputer -Filter *
```

Then we use that name in the following command to change its OU:
```powershell
Move-ADObject `
  -Identity "CN=WIN11-CLIENT,CN=Computers,DC=lab,DC=local" `
  -TargetPath "OU=Workstations,DC=lab,DC=local"
```

## 6. Final Test
To test everything is working as expected we can login on the workstation as Alice. At the Windows login screen we enter Alice's credentials and a successful login confirms:
- DNS working
- Workstation trusted the domain
- AD authentication functioned end-to-end
