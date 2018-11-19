keywords: SEHException, AppRunner, Crash, System.Interop.Services.SEHException, SEH, Exception,The process was terminated due to an unhandled exception

## Introduction
We've recently received reports from clients that experience intermittent crashes of .NET applications, when running from a network location or a mapped drive, possibly under Remote Desktop session.

## Symptoms
When you run the application from a mapped drive or a network location, the application crashes during execution.
The error details, which can be found the in Windows Event Viewer are similar to the following:
```csdiff
Framework Version: v4.0.30319
Description: The process was terminated due to an unhandled exception.
Exception Info: System.Runtime.InteropServices.SEHException
```

## The Cause
This is a known Microsoft issue in Windows Server 2008 and Windows Server 2012, as described in [this article](https://support.microsoft.com/en-gb/help/2536487/applications-crash-or-become-unresponsive-if-another-user-logs-off-a-r)


## Our Solution
We've developed a small utility called AppRunner, which allows you to run the application from a network location, without having this issue.
You can download AppRunner from [this link](https://github.com/FireflyMigration/AppRunner/releases)

For details about how to use it please refer to the [ReadMe page](https://github.com/FireflyMigration/AppRunner/blob/master/README.md)

## Additional Solution
I would like to propose another solution for this problem, suggested by our IT department. It is short and easy to implement by using windows command tool (CMD). Since we implemented it, three months ago, we had no related crashes. 
It works locally for the programmer or added to  the launching cmd file on the Citrix server for end users.
Example:
The application exe file is on the following path:  v:\VersionXYZ\myExe.exe
v - Is a network directory used by all the users to run the application which causes it to crash occasionally.
The solution we used is based on forcing the OS to treat drive V as a local directory on drive C by using MKLINK command.

Implementation:
•	Open CMD
•	Command: Cd c:
•	Command: mklink /D "location:\link" "Target directory - containing the exe folder or cmd to run".
Or, Detailed:
Create a  linked shortcut, named "V" to this location: \\files01\TheApplicationOrCMDFileDir
C:\> mklink /D C:\v \\files01\TheApplicationOrCMDFileDir

/D – Is a parameter represents directory.

Now, you can see on drive C a new linked shortcut to drive V. 


We implemented it for the users by changing the application launcher CMD\BAT file and added the line:
If not exist c:\v mklink /D C:\v \\files01\net

The whole CMD/BAT file looks like this:
"
...
If not exist c:\v mklink /D C:\v \\files01\TheApplicationOrCMDFileDir

c:
cd v\
cd VersionXYZ
start myExe.exe /ini=...
...
"

It works :-)
