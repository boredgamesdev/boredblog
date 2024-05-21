---
author: ["Ruben Whitney"]
title: "WMIEXEC"
date: "2024-03-10"
description: "Sample article showcasing basic code syntax and formatting for HTML elements."
summary: "Sample article showcasing basic code syntax and formatting for HTML elements."
tags: ["markdown", "syntax", "code", "gist"]
categories: ["themes", "syntax"]
series: ["Themes Guide"]
ShowToc: true
TocOpen: true
---

wmiexec is a popular tool in a red teamers and pentester playbook, I am interested in how it works under the hood. here is the description from the github page

"In this approach, we're looking at something akin to smbexec but with command execution via WMI. The primary perk is that it operates under the user account (which needs to be Admin), as opposed to SYSTEM. Additionally, it avoids generating noisy event log messages like smbexec.py does when creating a service. However, the downside is that it requires DCOM, meaning access to DCOM ports on the target machine is necessary."

define wmi
"Windows Management Instrumentation (WMI) is the infrastructure for management data and operations on Windows-based operating systems."

define dcom
"Specifies the Distributed Component Object Model (DCOM) Remote Protocol, which exposes application objects via remote procedure calls (RPCs) and consists of a set of extensions layered on the Microsoft Remote Procedure Call Extensions."

1. class is created with self
2. checks if smb is kerberos or not
3. decide with smb dialect to use
4. build DCOM connection with authentication

break down DCOM connection

These are used to login to WMI
```
from impacket.dcerpc.v5.dcomimport wmi
iInterface = dcom.CoCreateInstanceEx(wmi.CLSID_WbemLevel1Login, wmi.IID_IWbemLevel1Login)
iWbemLevel1Login = wmi.IWbemLevel1Login(iInterface)
```

next the NTLM connection to //./root/cimv2
Putting it all together, `//./root/cimv2` refers to the `cimv2` namespace on the local computer within the WMI framework. This namespace contains a variety of classes and objects representing information about the local system, hardware, and software configuration. It's commonly used for querying system-related data using WMI queries or scripts.

The code snippet `iWbemServices.GetObject('Win32_Process')` is a call to the `GetObject` method of the `iWbemServices` interface in the Windows Management Instrumentation (WMI) API.

 
 this uses the win32 process object to run commands
 self.shell = RemoteShell(self.__share, win32Process, smbConnection, self.__shell_type, silentCommand)

so SMB>DCOM>WMI>Win32Process>CMD/Powershell

CIMv2 is the main namespace used to start a remote shell, is there a way to use a different one? Cli jumps out as a possible candidate.

lets add a line to wmiexec to execute a remote process other than cmd
