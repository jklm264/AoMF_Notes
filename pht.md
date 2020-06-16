# Processes, Handles, and Tokens

- [Critical System Processes](#Critical-System-Processes)
- [Detect hidden processes](#Alternate-Process-Listing) - AoMF: "Realistically, it’s far easier to just inject code into a process that’s not hidden"
- [Link processes to user accounts](#Finding-SIDs)
- [Investigate lateral movement](#Detecting-Lateral-Movement)
- [Analyze PrivEsc Attacks](#Detect-privilege-escalation)



"This chapter combines three of the most common initial steps in an investigation"

1. What applications are running
2. What they're doing
3. What privilege level they have



## Processes

- `_EPROCESS` is a structure that represents a process.
  - Starts with the `Pcb` (process control block, aka `_KPROCESS`) which includes fields for address translation and "work time"
  - `ActiveProcessLinks` - most APIs rely on walking this doubly-linked list of active processes. This is a `_LIST_ENTRY` struct.
    - Other `_LIST_ENTRY` structs include: `SessionProcessLinks`, `JobLinks`, `ThreadListHead`, `MmProcessLinks`.
  - `PVOID Session` - stores information on user's logon session and GUI
  - `ImageFileName`
  - `ActiveThreads` - if zero, proc has exited. Can also corroborate with `ExitTime`.
  - `Peb` (Process Environment Block) exists in kernel point and points to an address in user mode. Contains lots of useful information including cwd, proc args, and many more!
  - `VadRoot` is the root node of the VAD tree including the original access permissions (RWX)

<img src="AoMF.Pictures/processresource.png" />

- **Handle table** is file descriptor table in *nix systems.

- Primary way Windows enforces security is with SID's



- Each proc has its own private virtual memory. Inside is:
  - the executable
  - list of loaded modules (DLLs and shared libraries)
  - stacks, heaps, .ro, user inputs, etc..
  - application-specific data structures (e.g. SQL Tables, config files, etc.)
- Windows organizes memory regions using virtual address descriptors (VADs)



### Process Organization

`_EPROCCESS` contains a [`_LIST_ENTRY`](datastructures.md#Embedded-Doubly-Linked-List)

- Tools like Process Explorer or Task Manager rely on walking this list via the ` NtQuerySystemInformation` API.

  

### Enumerating Processes in Memory

Vol.py steps when to list processes:

1. Locates kernel debugger data block (aka `_KDDEBUGGER_DATA64`)
2. Access ` PsActiveProcessHead` member which points to the head of the `EPROCESS` structures



Another way is to do the pool-scanning approach [(by their tags)](windowsmem4n6.md#Pool-Tag-Scanning). If we recall, this method of walking these `_LIST_ENTRY`s is not advisable as it does **NOT** find non-active processes since they would have been freed from the list.



### Critical System Processes

[See Forensics Class slides 7-11](https://drive.google.com/file/d/1G6Qu0cgU9pYkiwU1quXQof2VLctqTLZv/view?usp=sharing)

1. `Idle` and `System`
   - **Not real processes as in they do not have corresponding**
     **executable on disk.**
   - Idle is just a container that the kernel uses to charge CPU time
     for idle threads. 
   - System serves as the default home for threads that run in kernel mode. Thus, the System process (PID 4) appears to own any sockets or handles to files that kernel modules open executables.
2. `Csrss.exe` 
   - Plays a role in creating and deleting processes and threads. It maintains a private list of the objects that you can use to cross-reference with other data sources. **Expect to see multiple CSRSS processes because each session gets a dedicated copy; however, watch out for attempts to exploit the naming convention (csrsss.exe or cssrs.exe). The real one is located in the system32 directory.**
3. `Services.exe`
   - Manages Windows services and maintains a list of such services in its private memory space. **This process should be the parent for any svchost.exe (service host) instances** 
   - **It should be running from the system32 directory.**
4. `Svchost.exe`
   - Provides a container for DLLs and implement services.
   - Will have multiple *shared* svchost processes
5. `Lsass.exe`
   - The *local security authority subsystem* process is responsible for enforcing the security policy, verifying passwords, and creating access tokens. 
   - Plaintext password hashes can be found in its private memory space. 
   - There should be only one instance of lsass.exe running from the system32 directory, and its parent is winlogon.exe on pre-Vista machines, and wininit.exe on Vista and later systems. 
   - Stuxnet created two fake copies of lsass.exe , which caused them to stick out like a sore thumb.
6. `Winlogon.exe`
   - This process presents the interactive logon prompt, initiates the screen saver when necessary, helps load user profiles, and responds to Secure Attention Sequence (SAS) keyboard operations such as CTRL+ALT+DEL . 
   - Also, this process monitors files and directories for changes on systems that implement Windows File Protection (WFP). As with most other critical processes, its executable is located in the system32 directory.
   - Similar to what `session` contains ([from above](#Processes))
7. `Explorer.exe`
   - **One process for each logged-on user.** 
   - Handles GUI-based folder navigation, presenting the start menu, and so on. 
   - It also has access the documents you open and credentials you use to log in to FTP sites via Windows Explorer.
8. `Smss.exe`
   - The *session manager* is the first real user-mode process that starts during the boot sequence. 
   - It creates the sessions that isolate OS services from the various users who may log on via the console or Remote Desktop Protocol (RDP). 
   - When either Winlogon or Csrss ends normally, the system shuts down; if it happens unexpectedly, Smss.exe causes the system to stop responding (hangs).

### Analyzing Process Activity

1. `$vol.py pslist`

   - Walks the `_LIST_ENTRY` ([see above](#Process-Organization))

2. `$vol.py pstree`

   - Takes output of *pslist* and puts in tree view.

3. `$vol.py psscan` 

   - Pool-scans `_EPROCCESS` object instead of just walking the `_LIST_ENTRY`
   - [Also here from my notes](windowsmem4n6.md#Building-a-Pool-Scanner)
   - Can make process tree visualization
     - Suggested method is: `$vol.py psscan \--output=dot  \-\-output-file=processes.dot` then open in [Graphviz](

4. `$vol.py psxview`

   - Cross ('x') referencing alternate sources to find discrepancies.

   - [See seven sins of memory forensics](#Alternate-Process-Listing)

     

The following are other ways to find processes in a memory dump.



### Detecting DKOM Attacks

 **Direct Kernel Object Manipulation**

- Hide a process by unlinking its entry form the `_LIST_ENTRY` by simply overwriting the Flink and Blink pointers so they  point *around* the `_EPROCCESS` object.
  - Vol.py pslist is susceptible to this attack! **This is why it's better to use psscan or just psxview.**

- Malware can modify kernel objects by:
  - Loading a kernel driver, which has unrestricted access to objects in kernel memory
  - Mapping a writable view of the `\Device\PhysicalMemory` object
  - Using ` ZwSystemDebugControl()` API

#### The Case of Prolaco

- Used psscan to find hidden processes dropped from malware via the `ZWSystemDebugControl()` API
- Nice trick to show differences in pslist and psscan commands: `$ cat pslist.txt psscan.txt | awk '{print $2"\t"$3}' | sort | uniq –c | grep –v “ 2”`
- [Prolaco memory analysis by dtic.mil](https://apps.dtic.mil/dtic/tr/fulltext/u2/1004197.pdf)



#### Alternate Process Listing

1. Walk active process (`ActiveProcessLinks` ) via`_LIST_ENTRY`; using *pslist*
2. Process object scanning - pool scanning
3. Thread scanning `_ETHREAD`
4. CSRSS Handle Table - critical system process creates every process and thread so just walk the handle table ([referenced here in my notes](#Processes))
5. PspCid Table - Special handle table located in kernel memory that stores references to all **ACTIVE** process and thread objects. It's a member of `_KDDEBUGGER__DATA`.
6. Session processes - A member of `_EPROCCESS` that associates processes to which user's logon session.
   - Since APIs don't depend on this, attackers won't usually change it.
7. Desktop Threads - part of `tagDesktop` struct which store all threads attached to each desktop and thereby map a thread back to its owning proc.



`vol.py psxview` enumerates all of the Alternate Process Listings directly above. If you supply the `--apply-rules` option, you might also see *Okay* in the columns, which indicates that although the process was not found, it meets one of the valid exceptions described in the following list:

AoMF: "Realistically, it’s far easier to just inject code into a process that’s not hidden"



## Process Tokens

page 164 and slide 190

A process' token describes its security context. The tokens include Security Identifiers (SID) of user/group a given proc is running as AND its privilege's (specific tasks).

With proc tokens you can:

###### Map SIDs to Users Names

​	by taking the SID value and resolve it into the user/group name

###### Detect Lateral Movement

​	with artifacts left behind by hacking techniques like pass-the-hash that alter security contexts to something like Domain or Enterprise Admin.

###### Profile Process Behaviors

​	A privilege (the right to perform a specific task) is always present and enabled in a token in order for the proc to perform given tasks. So a proc's token-privilege contains many artifacts.

###### Detect Privilege Escalation

​	Since tools like Process Explorer can miss priv manipulation, memory forensics must be done to check.



### Data Structure _TOKEN

- very large structure that changes with each update. As of Windows 2003:
  - `UserAndGroups` and `UserAndGroupCount` (stores size) is an array of `_SID_AND_ATTRIBUTES` pointers to a `_SID` structure which contains `IdentifierAuthority` and `SubAuthority` which combines into the SID; *Ex: S-1-5-[snip]*.
  - `Privileges` and `PrivilegeCount` 
    - **Win XP and 2003:** array of `_LUID_AND_ATTRIBUTES` struct that lists and describes each privilege and attribute (whether it is *present*, *enabled*, *enabled by default*).
    - **Win Vista and later:** an instance of `_SEP_TOKEN_PRIVILEGES` with 64-bit attribute (see above) values. Bit positions correspond to particular privileges being *on* or *off*.



### Accessing Tokens from a Live System

- Many ways via APIs and vol.py

- Can also use GUI [Sysinternals Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer)

<img src="pht.Pictures/image-20200615145146421.png" alt-text="Sytsinternals proc explorer" height=400 width=500/>

#### Finding SIDs

SIDs are prefixed and then their remainder (of the SID) is composed of the SHA1 hash of the service in uppercase Unicode. [More info here](https://volatility-labs.blogspot.com/2012/09/movp-23-event-logs-and-service-sids.html)

<img src="pht.Pictures/image-20200615150758762.png" alt-text="SID breakdown" height=150 width=570 />

- Via cmd.exe:
  - **To enumerate SID or privileges:** `OpenProcessToken` API -> `GetTokenInformation()`
    - Can also query or set tokens of other users' and system procs *with admin access*
  - **To convert the `_SID` (S-1-5-[snip]) into human readable**: `ConvertSidToStringSid` API
  - **To return an account name from given SID**: use `LookupAccountSid`
    - `$vol.py getsids` will do the same. It will find each procs' token, tract the numerical components of the `_SID` struct, and translate them into strings. Then it will map the strings to user and group names on the local computer or domain.
- Via the registry:
  - ` $vol.py -f memory.img --profile=Win7SP0x86 printkey -K "Microsoft\Windows NT\CurrentVersion\ProfileList\S-1-5-21-4010035002-774237572-2085959976-1000"`
    - Will yield *ProfileImagePath* value to give you canonical username



### Detecting Lateral Movement

As seen in [Solving GrrCon Network Forensics Challenge with Volatility](https://volatility-labs.blogspot.com/2012/10/solving-grrcon-network-forensics.html)

1. Use *getsids* plugin to associate a proc with a user account
2. Found explorer.exe for current logged on user. A certain SID (ending 1115) doesn't display an account name.
   - **NOTE**: This is usually because vol.py doesn't have access to the remote machine's registry to do the lookup.

#### Privileges

A privilege is the permission to perform a specific task, such as debugging a process, shutting down the computer, changing the time zone, or loading a kernel driver.

- Before a process can enable a privilege, it must be present in the proc's token
- Admins can decide privileges by configuring the Local Security Policy (LSP) [aka SecPol.msc] or via `LsaAddAccountRights` API

#### Commonly Exploited Privileges

- To enable privs:
  - **Enabled by default:** via LSP, can specify certain privs enabled on proc start.
  - **Inheritance:** child procs inherit the *security context* of their parent.
  - **Explicit enabling:** via `AdjustTokenPrivileges` API
- Privs to be aware of:
  - `SeBackupPrivilege`: This grants read access to any file on the file system, regardless of its specified access control list (ACL). Attackers can leverage this privilege to copy locked files.	
  - `SeDebugPrivilege`: This grants the ability to read from or write to another process’ private memory space. It allows malware to bypass the security boundaries that typically isolate processes. Practically all malware that performs code injection from user mode relies on enabling this privilege. 
  - `SeLoadDriverPrivilege`: This grants the ability to load or unload kernel drivers.
  - `SeChangeNotifyPrivilege`: This allows the caller to register a callback function that gets executed when specific files and directories change. Attackers can use this to determine immediately when one of their configuration or executable files are removed by antivirus or administrators.
  - `SeShutdownPrivilege`: This allows the caller to reboot or shut down the system. Some infections, such as those that modify the Master Boot Record (MBR) don’t activate until the next time the system boots. Thus, you’ll often see malware trying to manually speed up the procedure by invoking a reboot. 

#### Analyzing Explicit Privs

198 or page 172





















## Process Handles

page 176 or 202

## Enumerating Handles in Memory



end page 187