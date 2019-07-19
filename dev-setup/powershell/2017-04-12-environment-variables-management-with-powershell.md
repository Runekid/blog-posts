---
layout: post
author: Wouter Van Schandevijl
title:  "Managing Environment Variables with PowerShell"
date:   2017-04-12 15:00:00 +0200
img:
  url: environment-variables.jpg
categories: dev-setup
tags: [powershell,windows]
extras:
  - githubproject: https://github.com/Laoujin/dotfiles/blob/master/config/PowerShell/scripts/envpath.ps1
    githubtext: The full ps1 source
toc:
  title: Environment vars
  icon: windows
last_modified_at: 2018-08-25 00:00:00 +0200
updates:
  - date: 2018-08-25 00:00:00 +0200
redirect_from: /blog/productivity/environment-variables-management-with-powershell/
---

Working with environment variables in Windows is as easy as:  
```
Win + Pause > "Advanced system settings" > "Environment Variables..."
```
After which you get a tiny, unresizable, form where you can view and manage them.
Something better eventually arrived with Windows 10 but still, PowerShell :)

Use Autohotkey to open the window with `Left Alt + Pause`:  
```autohotkey
LAlt & Pause::Run % "rundll32 sysdm.cpl,EditEnvironmentVariables"
```

<!--more-->

If you'd like a GUI instead you could try:
{% include github-stars.html url="rix0rrr/WindowsPathEditor" %}


# Getting environment variables

```powershell
# Listing all environment variables
Get-ChildItem Env:
env

# Or just those containing path
Get-ChildItem Env:*PATH* | Format-List

# Display a specific one (case insensitive)
$env:PATH
[environment]::GetEnvironmentVariable("PATH", "Process")

# They are the union of
# Machine: HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment
[environment]::GetEnvironmentVariable("PATH", "Machine")
# User: HKEY_CURRENT_USER\Environment
[environment]::GetEnvironmentVariable("PATH", "User")
```

In cmd.exe, all envs can be listed with `set`.


* * *


# Managing environment variables

```powershell
# Managing current session environment
$env:test = "value"
Remove-Item Env:test

# Persistent registry management (registry)
[Environment]::SetEnvironmentVariable("yaye", "value", "User")
[Environment]::SetEnvironmentVariable("yaye", "$null", "User")

# Requires RunAs Admin
[Environment]::SetEnvironmentVariable("yaye", "value", "Machine")
```

When using the PowerShell `$env:key = "val"`, the environment changes are available only for the current session.
When using the .NET `[Environment]::SetEnvironmentVariable`, the changes are persisted to the registry
but they do not become available in the current session.

A helper function to the rescue!

```powershell
Set-Alias se Set-Environment

function Set-Environment([String]$key, [String]$value) {
	[System.Environment]::SetEnvironmentVariable("$key", "$value", "User")
	Set-Item -Path Env:$key -Value $value
}
```

Already opened processes, tabs, etc won't see these environment changes.
The environment can be refreshed from the registry however.

```powershell
Set-Alias re Refresh-Environment

function Refresh-Environment {
	$locations = 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment', 'HKCU:\Environment'
	$locations | ForEach-Object {
		$k = Get-Item $_
		$k.GetValueNames() | ForEach-Object {
			$name = $_
			$value = $k.GetValue($_)
			Set-Item -Path Env:$name -Value $value
		}
	}

	# Put Machine and User $env:PATH together
	$machinePath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
	$userPath = [Environment]::GetEnvironmentVariable("PATH", "User")
	$Env:PATH = "$machinePath;$userPath"
}
```

If you have chocolatey installed, `RefreshEnv` will also work.


* * *


# $env:PATH helpers

Use `fp` to list all directories in `$env:path`.
It accepts a search needle as parameter. ex: `fp node`.

By using the PowerShell builtin variable `$args`, it is not required to
put quotes around directories containing spaces.

```powershell
Set-Alias fp Find-EnvironmentPath

function Find-EnvironmentPath {
	$needle = ($args -join " ").TrimEnd("\")
	$needle = [Regex]::Escape($needle)

	$env:PATH.split(";") |
		Where-Object { $_.TrimEnd("\") -match $needle } |
		Sort-Object |
		Get-Unique -AsString
}
```

Append a directory to the User path with `ap c:\bins`.

```powershell
Set-Alias ap Append-EnvironmentPath

function Append-EnvironmentPath {
	$toAppend = ($args -join " ").TrimEnd("\")
	if (-not (Test-Path $toAppend)) {
		Write-Host "Path doesn't exist..."
		Write-Host $toAppend
		return
	}
	$env:PATH = $env:PATH + ";$toAppend"
	$userPath = [Environment]::GetEnvironmentVariable("PATH", "User")
	[Environment]::SetEnvironmentVariable("PATH", "$userPath;$toAppend", "User")
}
```

Remove a directory from the User path with `delp c:\Program Files\someBin`.

```powershell
Set-Alias delp Remove-EnvironmentPath

function Remove-EnvironmentPath {
	$toRemove = ($args -join " ").TrimEnd("\")

	$userPath = [Environment]::GetEnvironmentVariable("PATH", "User")
	$changedUserPath = $userPath.Split(";") |
		ForEach-Object { $_.TrimEnd("\") } |
		Where-Object { $_ -ne $toRemove }

	$changedUserPath = $changedUserPath -Join ";"

	if ($userPath -ne $changedUserPath) {
		[Environment]::SetEnvironmentVariable("PATH", "$changedUserPath", "User")
		$env:PATH = [Environment]::GetEnvironmentVariable("PATH", "Machine")
		$env:PATH += ";$changedUserPath"
		return
	}

	echo "Not removed"

	$machinePath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
	$isInMachine = $machinePath.split(";") | Where-Object { $_.Trim("\") -eq $toRemove }
	if ($isInMachine) {
		echo "Is present in Machine scope"
	}
}
```


* * *


# Useful environment variables

Win + Pause: `Control Panel\System and Security\System`  
PowerShell: `$env:USERNAME`  
Cmd: `%USERNAME%`  

These are probably worth remembering.

```powershell
"$env:USERNAME@$env:USERDOMAIN"

$env:APPDATA
# C:\Users\Wouter\AppData\Roaming

$env:HOME
$env:USERPROFILE
# C:\Users\Wouter

$env:LOCALAPPDATA
# C:\Users\Wouter\AppData\Local

$env:TEMP
# C:\Users\Wouter\AppData\Local\Temp
```
