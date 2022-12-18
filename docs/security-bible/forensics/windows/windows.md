---
layout: default
title: Windows Endpoint
parent: Forensics
grand_parent: Security Bible
nav_order: 1
---

# Windows Endpoint Forensics
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Windows Endpoint Forensics

### Windows Processes
``` yaml
# System 
hosts threads that only run in kernel mode.
Image: %Systemroot%\System32\ntoskrnl.exe (Task Manager / Process Hacker) and None (Process Explorer)

# ntoskrnl.exe 
Windows NT operating system kernel executable. Acts as the central module (kernel) of the Windows OS.

# smss.exe 
Sessions manager process is the first user mode process. It’s responsible for creating the list of 
the environment variables, the security descriptors that’ll be used by the various system resources, 
initializing the rest of the registry (HKLM Software and the SAM and security hives) and a lot more.

- The system process creates the first SMSS instance and its known as the master SMSS.EXE process. 
It’s the only instance of SMSS.EXE that’ll stay after windows has booted up and one of the characteristics 
of this process is that it doesn’t have any command line arguments.
- The master SMSS process will create at least two instances of itself. One in “Session 0” (OS) that 
will be responsible for creating the “wininit.exe” process and another in “Session 1” (User) that’ll 
represent the first logged-on user and will create the “winlogon.exe”. Both will spawn a “csrss.exe” process.
- All the children SMSS.EXE processes created by the master SMSS.EXE process will exit after finishing their JOB.

# Wininit.exe
Windows initialization process is responsible for initializing and setting a lot of things.
- It sets the default environment variables (USERPROFILE, ALLUSERPROFILE, PUBLIC and ProgramData).
- It creates the LSASS process and sets the LSA encryption key.
- It creates the Services Control manager by launching the SERVICES.EXE process
- It creates the temp directory in the system root (%Systemroot%\Temp).

# regsvr32.exe 
Is a command-line utility to register and unregister OLE controls, such as DLLs and ActiveX controls 
in the Windows Registry. Regsvr32.exe is installed in the %systemroot%\System32 folder in 
Windows XP and later versions of Windows.

Usage:
$ Regsvr32 [/u] [/n] [/i[:cmdline]] <dllname>
- /u - Unregister server
- /i - Call DllInstall passing it an optional [cmdline]; when it is used with /u, it calls dll uninstall
- /n - do not call DllRegisterServer; this option must be used with /i
- /s – Silent; display no message boxes

# rundll32.exe
Loads and runs 32bit dynamic-link libraries (DLLs). 
There are no configurable settings for Rundll32.


```

### Windows Evidence of File and Folder Opening

``` yaml
# Open/Save MRU
Registry key tracks files that have been opened or saved within a Windows shell dialog box 
including web browsers and most applications. 
Interpretation:
- The “*” key – This subkey tracks the most recent files of any extension input in an OpenSave dialog
- .??? (Three letter extension) – This subkey stores file info from the OpenSave dialog by specific extension
Location (Win7+): NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\
OpenSavePIDlMRU

# Last-Visit MRU
Tracks exe used by application to open files documented in Open/Save MRU the dir location 
for the last file that was accessed by that app.
Interpretation: We get two important pieces of information from this key
- applications executed by the user 
- and the last place in the file system that those applications interacted with.
- Interesting and hidden directories are often identified via this registry key.
Location (win7+): NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU

# Recent Files
Registry key tracks the last files and folders opened and is used to populate data 
in ‘recent’ menus. 
Intepretation:
    RecentDocs: Tracks overall order of the last 150 files or folders opened.
    .???: These subkeys store the last 20 files opened by the user of each extension type.
    MRU list tracks the temporal order in which each file was opened. 
    The most recently used (MRU) item is associated with the last write time of the key, 
    providing one timestamp of file opening for each file extension type.
    Folder: This subkey stores the last 30 folders opened by the user. 
    The most recently used (MRU) item in this key is associated with the last write 
    time of the key, providing the time of opening for that folde
Location: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs

# Jump Lists
Allows users to access frequently used items.
Data stored in AutomaticDestinations folder will each have a unique file prepended with the 
AppID of the association application and embedded with LNK files in each stream.
Interpretation:
- Each jump list file is named according to an application identifier (AppID). 
    List of Jump List IDs -> https://dfir.to/EZJumpList
- Each Jump List contains a collection of items interacted with (up to ~2000 items per application)
- Each entry is represented as a LNK shell item providing additional data
    - Target Timestamps
    - File Size
    - Local Drive | Removable Media | Network Share Info
    - Entries kept in MRU order including a timestamp for each item
Location:  
- %USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations
- %USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations

# Shell Bags:
Which folders were accessed on the local machine, network or removable device.
Explorer access: USRCLASS.DAT
Desktop Access: NTUSER.DAT
Interpretation:
- Massive collection of data on folders accessed by each user
- Folder file system timestamps are archived in addition to first and last interaction times
- “Exotic” items recorded like mobile device info, control panel access, and Zip archive acces
Location:
    Primary Data
    - USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
    - USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU
    Residual Desktop Items and Network Shares
    - NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU
    - NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags

# Shortcut Files (LNK)
Shortcut files created for:
Shortcut files are automatically created by Windows, tracking files and 
folders opened by a user
Interpretation:
- Date/Time file of that name was first opened
    - Creation Date of Shortcut (LNK) File
- Date/Time file of that name was last opened
    - Last Modification Date of Shortcut (LNK) File
- LNK Target File (Internal LNK File Information) Data
    - Modified, Access, and Creation times of the target file
    - Volume Information (Name, Type, Serial Number)
    - Network Share information
    - Original Location
    - Name of System
Location:  
- Win7+: %USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\
- Win7+: %USERPROFILE%\AppData\Roaming\Microsoft\Office\Recent\

# IE|Edge file://
Internet Explorer History databases have long held information on local and remote file access (via network shares), 
giving us an excellent means for determining files accessed on the system, per user. 
Information can be present even on Win11+ systems missing the Internet Explorer application. 
Not just related to internet browsing but also stores local, removable and remote (via network shares) file access.
Interpretation:
- Entries recorded as: file:///C:/directory/filename.ext
- Does not mean file was opened in a browse
Location:
- IE10–11 & Win10+: %USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat

# Office Recent Files
MS Office apps will track their own recent file
Location: NTUSER.DAT

# MS Word Reading Locations
Starting with Word 2013, the last known position of the user within a Word document is recorded.
Interpretation:
- Another source tracking recent documents opened
- The last closed time is also tracked along with the last position within the file
- Together with the last opened date in the Office File MRU key, a last session duration can be determined
Location: NTUSER\Software\Microsoft\Office\<Version>\Word\Reading Location

```

### Windows Evidence of Application Execution

``` yaml
# Shimcache 
The Windows Application Compatibility Database is used by Windows to identify possible 
application compatibility challenges with executables. It tracks the executable file path 
and binary last modified time.
Interpretation:
Any executable present in the file system could be found in this key. Data 
can be particularly useful to identify the presence of malware on devices 
where other application execution data is missing (such as Windows servers). 
- Full path of executable
- Windows 7+ contains up to 1,024 entries (96 entries in WinXP)
- Post-WinXP no execution time is available 
- Executables can be preemptively added to the database prior to execution. 
- The existence of an executable in this key does not prove actual execution.
Location:
- Win7+: SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCach

# Windows 10 Timeline
Win10 records recently used applications and files in a “timeline” 
database in SQLite format.
Interpretation:
- Full path of executed application
- Start time, end time, and duration
- Items opened within application
- URLs visited
- Databases still present even after feature deprecation in late-Win10
Location:
- C:\Users\<profile>\AppData\Local\ConnectedDevicesPlatform\<account-ID>\ActivitiesCache.db

# Task Bar Feature Usage
Tracks how a user has interacted with the taskbar.
Interpretation:
- Only tracks GUI applications
- Does not include timestamps
- AppLaunch tracks data only for pinned applications, showing user knowledge of the application 
    - Data persists after an application is unpinned
- AppSwitched tracks a count of application focus, showing user interaction directed at the application
    - Not tied to pinned applications
Location:
- Win 10 1903+: NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\FeatureUsage

# BAM/DAM
Windows Background/Desktop Activity Moderator (BAM/DAM) is maintained by the Windows power 
management sub-system. (Available in Win10+)
Interpretation:
- Provides full path of file executed and last execution date/time
- Typically up to one week of data available
- “State” key used in Win10 1809+
Location:
- SYSTEM\CurrentControlSet\Services\bam\State\UserSettings\{SID}
- SYSTEM\CurrentControlSet\Services\dam\State\UserSettings\{SID

# Amcache.hve
Amcache tracks installed applications, programs executed (or present), drivers loaded, and more. 
What sets this artifact apart is it also tracks the SHA1 hash for executables and drivers. (Available in Win7+)
Interpretation:
- A complete registry hive, with multiple sub-keys
- Full path, file size, file modification time, compilation time, and publisher metadata
- SHA1 hash of executables and drivers
- Amcache should be used as an indication of executable and driver presence on the system, but not to prove actual execution
Location:
- C:\Windows\AppCompat\Programs\Amcache.hve

# System Resource Usage Monitor (SRUM)
SRUM records 30 to 60 days of historical system performance including applications run, user accounts responsible, 
network connections, and bytes sent/received per application per hour.
SRUDB.dat is an Extensible Storage Engine database 
Interpretation:
- Three tables in SRUDB.dat are particularly important:
   - {973F5D5C-1D90-4944-BE8E-24B94231A174} = Network Data Usage
   - {d10ca2fe-6fcf-4f6d-848e-b2e99266fa89} = Application Resource Usage
   - {DD6636C4-8929-4683-974E-22C046A43763} = Network Connectivity Usage
Location:
- Win8+: C:\Windows\System32\SRU\SRUDB.dat

# Jump Lists
Windows Jump Lists allow user access to frequently or recently used items quickly via the task bar. 
First introduced in Windows 7, they can identify applications in use and a wealth of metadata about items 
accessed via those applications
Interpretation: 
- Each jump list file is named according to an application identifier (AppID). 
List of Jump List IDs -> https://dfir.to/EZJumpList
- Automatic Jump List Creation Time = First time an item added to the jump list. 
Typically, the first time an object was opened by the application.
- Automatic Jump List Modification Time = Last time item added to the jump list. 
Typically, the last time the application opened an object. 
Location:
- %USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations

# Last Visited MRU
Tracks applications in use by the user and the directory location for the last file accessed by the application.
Interpretation:
- We get two important pieces of information from this key: 
   - applications executed by the user, 
   - and the last place in the file system that those applications interacted with. 
   Interesting and hidden directories are often identified via this registry key.
Location:
- Win7+: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU

# Prefetch
Designed to speed up applications’ startup processes.
File names consist of two parts: an executable name, and an eight-character hash of the executable’s location. 
Prefetch files contain various metadata: executable name, run count, volume information, files and directories referenced by the executable, and of course, timestamps. 
Interpretation: 
- file’s creation timestamp as the timestamp of the first execution. 
- embedded timestamp of the last execution
- since version 26 (Windows 8.1), the 7 most recent last run times.
Location: %SystemRoot%\Prefetch

# Commands Executed in the Run Dialog
A history of commands typed into the Run dialog box are stored for each user.
Interpretation:
- It is an MRU key, so it has temporal order via the MRUList key
Location:
- NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU

# UserAssist
UserAssist records metadata on GUI-based program executions.
Interpretation:
- GUIDs identify type of execution (Win7+)
   - CEBFF5CD Executable File Execution
   - F4E57C4B Shortcut File Execution
- Values are ROT-13 Encoded
- Application path, last run time, run count, focus time and focus count
Location:
- NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count
```
---