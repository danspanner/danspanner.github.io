---
layout: page
title:  Handy scripts and one-liners
---

This should probably live in a gist or something. But eh. This works too.

Just a collection of handy little bits I've collected for various problems I've been asked to solve time and time again.

*   [Powershell](#HandyScriptsandOne-liners-Powershell)
    *   [Set your execution policy](#HandyScriptsandOne-liners-Setyourexecutionpolicy)
    *   [Add "Features on Demand" in Windows 10 1809](#HandyScriptsandOne-liners-Add"FeaturesonDemand"inWindows101809)
    *   [Copy members of one AD group to another group](#HandyScriptsandOne-liners-CopymembersofoneADgrouptoanothergroup)
    *   [Find out what DC I'm talking to](#HandyScriptsandOne-liners-FindoutwhatDCI'mtalkingto)
    *   [Check to see if users have a blank pager field (and fix it if they do)](#HandyScriptsandOne-liners-Checktoseeifusershaveablankpagerfield(andfixitiftheydo))
    *   [Get a list of printers from the print server (with corresponding IP addresses)](#HandyScriptsandOne-liners-Getalistofprintersfromtheprintserver(withcorrespondingIPaddresses))
    *   [Find the remote logged in user](#HandyScriptsandOne-liners-Findtheremoteloggedinuser)
    *   [Find your PC serial number](#HandyScriptsandOne-liners-FindyourPCserialnumber)
    *   [Modify the property of a value stored in a variable](#HandyScriptsandOne-liners-Modifythepropertyofavaluestoredinavariable)
    *   [Check to see if a website is up or not](#HandyScriptsandOne-liners-Checktoseeifawebsiteisupornot)
    *   [Pulling the jpegPhoto attribute from an AD User](#HandyScriptsandOne-liners-PullingthejpegPhotoattributefromanADUser)
    *   [tcpdump without tcpdump](#HandyScriptsandOne-liners-tcpdumpwithouttcpdump)
    *   [Colin's pdf script](#HandyScriptsandOne-liners-Colin'spdfscript)
*   [bash](#HandyScriptsandOne-liners-bash)
    *   [Finding things in bash](#HandyScriptsandOne-liners-Findingthingsinbash)
    *   [Recursively grep](#HandyScriptsandOne-liners-Recursivelygrep)
    *   [snmpwalk of all printers](#HandyScriptsandOne-liners-snmpwalkofallprinters)
*   [Docker](#HandyScriptsandOne-liners-Docker)
    *   [Run something inside a container](#HandyScriptsandOne-liners-Runsomethinginsideacontainer)
    *   [Copying files into or out of containers](#HandyScriptsandOne-liners-Copyingfilesintooroutofcontainers)
*   [SQL queries etc.](#HandyScriptsandOne-liners-SQLqueriesetc.)
    *   [Connecting to mysql on sps-lweb01](#HandyScriptsandOne-liners-Connectingtomysqlonsps-lweb01)
    *   [Listing users](#HandyScriptsandOne-liners-Listingusers)
    *   [Search tables in MSSQL](#HandyScriptsandOne-liners-SearchtablesinMSSQL)
*   [Cisco CLI](#HandyScriptsandOne-liners-CiscoCLI)
    *   [Find a mac address](#HandyScriptsandOne-liners-Findamacaddress)
*   [Links](#HandyScriptsandOne-liners-Links)
    *   [Fix for not being able to see specific attributes in AD](#HandyScriptsandOne-liners-FixfornotbeingabletoseespecificattributesinAD)

A collection of quick powershell / bash scripts and links to make my life easier

Powershell
==========

### Set your execution policy

By default powershell will want you to have signed scripts. That's crazy talk, no one signs their scripts!

```Powershell
Set-ExecutionPolicy -Scope localmachine unrestricted
```

Run that in an admin shell to tell Windows to STFU.

### Add "Features on Demand" in Windows 10 1809

Microsoft changed some useful tooling in Win 10 1809, which is both good and bad. I found SCCM interfered with installing bits though.

First and foremost, see what Windows can add-

```powershell
Get-WindowsCapability -Online
```

Then have a trawl through the list and see what's useful to you. Minimum for a SysAdmin is probably

```Powershell
Add-WindowsCapability -Online -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0
Add-WindowsCapability -Online -Name Rsat.Dns.Tools~~~~0.0.1.0
Add-WindowsCapability -Online -Name Rsat.DHCP.Tools~~~~0.0.1.0
```

The same thing can be done with DISM, but why DISM when you can posh?

### Copy members of one AD group to another group

```powershell
Add-ADGroupMember -Identity 'New Group' -Members (Get-ADGroupMember -Identity 'Old Group' -Recursive)
```

So, for instance, for an "All Staff" group that needs to be copied, I used

```powershell
Add-ADGroupMember -Identity "New Group" -Members (Get-ADGroupMember "All Staff")
```

### Find out what DC I'm talking to

`nltest /DSGETDC:`

### Check to see if users have a blank pager field (and fix it if they do)

```powershell
$pagerusers = Get-ADUser -Filter \* -properties \* -SearchBase "OU=Users,DC=domain,DC=com" | where pager -eq $null
```

Then get the count with `$pagerusers.count`

```powershell
Get-ADUser -Filter \* -properties \* -SearchBase "OU=Users,DC=domain,DC=com" | where pager -eq $null | % {Set-ADUser $\_ -add @{Pager="1"}}
```

### Get a list of printers from the print server (with corresponding IP addresses)

This has a downside of needing admin access to run (or at least on my print server it did, I can't speak for your own permissions)

```powershell
Get-WmiObject win32\_printer -ComputerName printserver | select sharename,location,portname
```

### Find the remote logged in user

There are a few ways of doing this-

```WMIC /NODE: "hostname" COMPUTERSYSTEM GET USERNAME```

However I've found this can sometimes go awry and return an RPC server error. You can use the [Microsoft Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite), which comes with a tool called PsLoggedon.exe

```PsLoggedon.exe -l \\\\hostname```

### Find your PC serial number

```powershell
get-ciminstance win32\_bios | select serialnumber
```

### Modify the property of a value stored in a variable

Something I found from the AzureAD group script was it wasn't setting a particular value in the template it was attempting to push.

If we, for example, create a directory template-

**GroupFix.ps1**
```Powershell
$GroupName = "SYSAdmins"
$AllowGroupCreation = "False"

Connect-AzureAD

$settingsObjectID = (Get-AzureADDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id
if(!$settingsObjectID)
{
	  $template = Get-AzureADDirectorySettingTemplate | Where-object {$\_.displayname -eq "group.unified"}
    $settingsCopy = $template.CreateDirectorySetting()
    New-AzureADDirectorySetting -DirectorySetting $settingsCopy
    $settingsObjectID = (Get-AzureADDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id
}

$settingsCopy = Get-AzureADDirectorySetting -Id $settingsObjectID
$settingsCopy\["EnableGroupCreation"\] = $AllowGroupCreation

if($GroupName)
{
	$settingsCopy\["GroupCreationAllowedGroupId"\] = (Get-AzureADGroup -SearchString $GroupName).objectid
}

Set-AzureADDirectorySetting -Id $settingsObjectID -DirectorySetting $settingsCopy

(Get-AzureADDirectorySetting -Id $settingsObjectID).Values
```

For whatever reason this wasn't setting the value correctly in the variable `$settingsCopy` -

```powershell
$settingsCopy.Values

Name                          Value
----                          -----
CustomBlockedWordsList
EnableMSStandardBlockedWords  False
ClassificationDescriptions
DefaultClassification
PrefixSuffixNamingRequirement
AllowGuestsToBeGroupOwner     False
AllowGuestsToAccessGroups     True
GuestUsageGuidelinesUrl
GroupCreationAllowedGroupId
AllowToAddGuests              True
UsageGuidelinesUrl
ClassificationList
EnableGroupCreation
```

So I manually updated it with this-

```$settingsCopy\["EnableGroupCreation"\] = $False```

### Check to see if a website is up or not

```powershell
while ($true){Invoke-WebRequest https://pages.assessform.edu.au |select statuscode; start-sleep -seconds 2}
```

### Pulling the jpegPhoto attribute from an AD User

I went down a bit of a rabbit hole to see if I could reverse out the photo from AD.

[https://docs.microsoft.com/en-us/windows/win32/adschema/a-photo](https://docs.microsoft.com/en-us/windows/win32/adschema/a-photo)

Turns out, AD can store a user photo as a byte stream. In AD, if you view the attribute, you will see the raw hex of the photo. We can tell it's a JPEG from the FF D8

[What a JPEG looks like.](https://github.com/corkami/pics/blob/master/binary/JPG.png)

So how do we get it out? With our good friend system.io.file!

First, we'll store the properties for our user in a variable.

```powershell
$photo = get-aduser -Identity t.smith -Properties jpegPhoto
```

Then

```powershell
[system.io.file\]::writeallbytes('C:\\tmp\\output.jpg', $photo.jpegPhoto.value)
```

This is slightly annoying, as there are actually two attributes- `jpegPhoto` and `photo`. However one is actually a thumbnail, and can be retrieved slightly differently (or at least without as much fuss)

### tcpdump without tcpdump

You can use netsh!

```
netsh trace start scenario=netconnection capture=yes report=yes persistent=no maxsize=1024 correlation=yes tracefile=c:\\temp\\nettrace.etl
```

Don't forget to stop your trace

```netsh trace stop```

You can then look at your network trace in [Message Analyser](https://www.microsoft.com/en-us/download/confirmation.aspx?id=44226) if you so please. Doco for adding filters is here

[https://docs.microsoft.com/en-us/message-analyzer/applying-and-managing-filters](https://docs.microsoft.com/en-us/message-analyzer/applying-and-managing-filters)

### Copy and rename pdf script

Quick loop to copy and rename a PDF on the network drives.

```powershell
$students = Get-ChildItem . -Directory
foreach ($student in $students) {
	Copy-Item -Path ".\\<file name here>.pdf" -Destination ".\\$student\\<file name here> $student.pdf"
}
```

Just make sure you run it from the directory you're copying stuff from.

bash
====

### Finding things in bash

Since I always forget, this is a catch-all for finding things

```find /directory/to/search -name "name here"```

### Recursively grep

```grep -r "text here" /dir```

### snmpwalk of all printers

WaynSomeonee asked me for a list of printer makes and models we have around the place, which is something that you can't get from the print server (unfortunately). However, we can get it through the wonders of SNMP.

I pulled the list of IPs from the print server (see the powershell section), then whipped up a quick and dirty bash script.

```bash
#!/bin/bash

cat /mnt/c/Users/d.mackie/Printer\_ips.txt | while read output
do
        echo "$output"
        snmpwalk -v 2c -c public "$output" Printer-MIB::prtGeneralPrinterName.1 && snmpwalk -v 2c -c public "$output" Printer-MIB::prtInputVendorName.1.1
done
```

The output looks like garbage and I had to clean it up a little, but it's relatively serviceable for 20 minutes work.

Docker
======

### Run something inside a container

Cos I always forget the damn syntax

```docker exec -ti container <program name here>```

### Copying files into or out of containers

Even when a container is offline we can pull or update files from it-

```docker cp container:/path/to/file /path/to/local/storage```

And the reverse to copy a file into a container

```docker cp /file/to/copy container:/file/to/replace```

SQL queries etc.
================

### Search tables in MSSQL

```sql
SELECT      c.name  AS 'ColumnName'
            ,t.name AS 'TableName'
FROM        sys.columns c
JOIN        sys.tables  t   ON c.object\_id = t.object\_id
WHERE       c.name LIKE '%MyName%'
ORDER BY    TableName
            ,ColumnName;
```
  

Cisco CLI
=========

### Find a mac address

`show mac address-table`

You can also add add the MAC at the end (if known) if you're searching for a specific device. Additionally you can search for a specific MAC in the table

`show mac address-table address 805e.c031.d602`

Fix for not being able to see specific attributes in AD
-------------------------------------------------------

[https://serverfault.com/questions/151919/grant-account-write-access-to-specific-attributes-on-active-directory-user-objec](https://serverfault.com/questions/151919/grant-account-write-access-to-specific-attributes-on-active-directory-user-objec)
