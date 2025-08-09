# SMB
Server Message Block (SMB) is a client-server protocol that regulates access to files and entire directories and other network resources such as printers, routers, or interfaces released for the

network. Information exchange between different system processes can also be handled based on the SMB protocol. SMB first became available to a broader public, for example, as part of the OS/2 network

operating system LAN Manager and LAN Server. Since then, the main application area of the protocol has been the Windows operating system series in particular, whose network services support SMB in a

downward-compatible manner - which means that devices with newer editions can easily communicate with devices that have an older Microsoft operating system installed. With the free software project

Samba, there is also a solution that enables the use of SMB in Linux and Unix distributions and thus cross-platform communication via SMB.

The SMB protocol enables the client to communicate with other participants in the same network to access files or services shared with it on the network. The other system must also have implemented

the network protocol and received and processed the client request using an SMB server application. Before that, however, both parties must establish a connection, which is why they first exchange

corresponding messages.

In IP networks, SMB uses TCP protocol for this purpose, which provides for a three-way handshake between client and server before a connection is finally established. The specifications of the TCP

protocol also govern the subsequent transport of data. We can take a look at some examples here.

An SMB server can provide arbitrary parts of its local file system as shares. Therefore the hierarchy visible to a client is partially independent of the structure on the server. Access rights are

defined by Access Control Lists (ACL). They can be controlled in a fine-grained manner based on attributes such as execute, read, and full access for individual users or user groups. The ACLs are

defined based on the shares and therefore do not correspond to the rights assigned locally on the server.

# Samba

As mentioned earlier, there is an alternative implementation of the SMB server called Samba, which is developed for Unix-based operating systems. Samba implements the Common Internet File System

(CIFS) network protocol. CIFS is a dialect of SMB, meaning it is a specific implementation of the SMB protocol originally created by Microsoft. This allows Samba to communicate effectively with newer

Windows systems. Therefore, it is often referred to as SMB/CIFS.

However, CIFS is considered a specific version of the SMB protocol, primarily aligning with SMB version 1. When SMB commands are transmitted over Samba to an older NetBIOS service, connections

typically occur over TCP ports 137, 138, and 139. In contrast, CIFS operates over TCP port 445 exclusively. There are several versions of SMB, including newer versions like SMB 2 and SMB 3, which

offer improvements and are preferred in modern infrastructures, while older versions like SMB 1 (CIFS) are considered outdated but may still be used in specific environments.

```c
SMB Version 	Supported 	Features
CIFS 	Windows NT 4.0 	Communication via NetBIOS interface
SMB 1.0 	Windows 2000 	Direct connection via TCP
SMB 2.0 	Windows Vista, Windows Server 2008 	Performance upgrades, improved message signing, caching feature
SMB 2.1 	Windows 7, Windows Server 2008 R2 	Locking mechanisms
SMB 3.0 	Windows 8, Windows Server 2012 	Multichannel connections, end-to-end encryption, remote storage access
SMB 3.0.2 	Windows 8.1, Windows Server 2012 R2 	
SMB 3.1.1 	Windows 10, Windows Server 2016 	Integrity checking, AES-128 encryption
```

With version 3, the Samba server gained the ability to be a full member of an Active Directory domain. With version 4, Samba even provides an Active Directory domain controller. It contains several

so-called daemons for this purpose - which are Unix background programs. The SMB server daemon (smbd) belonging to Samba provides the first two functionalities, while the NetBIOS message block daemon

(nmbd) implements the last two functionalities. The SMB service controls these two background programs.

We know that Samba is suitable for both Linux and Windows systems. In a network, each host participates in the same workgroup. A workgroup is a group name that identifies an arbitrary collection of

computers and their resources on an SMB network. There can be multiple workgroups on the network at any given time. IBM developed an application programming interface (API) for networking computers

called the Network Basic Input/Output System (NetBIOS). The NetBIOS API provided a blueprint for an application to connect and share data with other computers. In a NetBIOS environment, when a machine

goes online, it needs a name, which is done through the so-called name registration procedure. Either each host reserves its hostname on the network, or the NetBIOS Name Server (NBNS) is used for this

purpose. It also has been enhanced to Windows Internet Name Service (WINS).

# Default Configuration

```c
cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no

```

We see global settings and two shares that are intended for printers. The global settings are the configuration of the available SMB server that is used for all shares. In the individual shares,

however, the global settings can be overwritten, which can be configured with high probability even incorrectly. Let us look at some of the settings to understand how the shares are configured in

Samba.

```c
Setting 	Description
[sharename] 	The name of the network share.
workgroup = WORKGROUP/DOMAIN 	Workgroup that will appear when clients query.
path = /path/here/ 	The directory to which user is to be given access.
server string = STRING 	The string that will show up when a connection is initiated.
unix password sync = yes 	Synchronize the UNIX password with the SMB password?
usershare allow guests = yes 	Allow non-authenticated users to access defined share?
map to guest = bad user 	What to do when a user login request doesn't match a valid UNIX user?
browseable = yes 	Should this share be shown in the list of available shares?
guest ok = yes 	Allow connecting to the service without using a password?
read only = yes 	Allow users to read files only?
create mask = 0700 	What permissions need to be set for newly created files?
```

# Dangerous Settings

Some of the above settings already bring some sensitive options. However, suppose we question the settings listed below and ask ourselves what the employees could gain from them, as well as

attackers. In that case, we will see what advantages and disadvantages the settings bring with them. Let us take the setting browseable = yes as an example. If we as administrators adopt this

setting, the company's employees will have the comfort of being able to look at the individual folders with the contents. Many folders are eventually used for better organization and structure. If

the employee can browse through the shares, the attacker will also be able to do so after successful access.

```c
Setting 	Description
browseable = yes 	Allow listing available shares in the current share?
read only = no 	Forbid the creation and modification of files?
writable = yes 	Allow users to create and modify files?
guest ok = yes 	Allow connecting to the service without using a password?
enable privileges = yes 	Honor privileges assigned to specific SID?
create mask = 0777 	What permissions must be assigned to the newly created files?
directory mask = 0777 	What permissions must be assigned to the newly created directories?
logon script = script.sh 	What script needs to be executed on the user's login?
magic script = script.sh 	Which script should be executed when the script gets closed?
magic output = script.out 	Where the output of the magic script needs to be stored?
```

Let us create a share called [notes] and a few others and see how the settings affect our enumeration process. We will use all of the above settings and apply them to this share. For example, this

setting is often applied, if only for testing purposes. If it is then an internal subnet of a small team in a large department, this setting is often retained or forgotten to be reset. This leads to

the fact that we can browse through all the shares and, with high probability, even download and inspect them.


# Restart Samba

```c
sudo systemctl restart smbd
```

# SMBclient - Connecting to the Share

```c
smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

We can see that we now have five different shares on the Samba server from the result. Thereby print$ and an IPC$ are already included by default in the basic setting, as we have already seen. Since

we deal with the [notes] share, let us log in and inspect it using the same client program. If we are not familiar with the client program, we can use the help command on successful login, listing

all the possible commands we can execute.

```c
smbclient //10.129.14.128/notes

Enter WORKGROUP\<username>'s password: 
Anonymous login successful
Try "help" to get a list of possible commands.


smb: \> help

?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!            


smb: \> ls

  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

# Download Files from SMB
```c
smb: \> get prep-prod.txt 

getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec) 
(average 8,7 KiloBytes/sec)


smb: \> !ls

prep-prod.txt


smb: \> !cat prep-prod.txt

[] check your code with the templates
[] run code-assessment.py
[] …	
```

# Samba Status
```c
root@samba:~# smbstatus

Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           

No locked files
```

From the administrative point of view, we can check these connections using smbstatus. Apart from the Samba version, we can also see who, from which host, and which share the client is connected.

This is especially important once we have entered a subnet (perhaps even an isolated one) that the others can still access.

For example, with domain-level security, the samba server acts as a member of a Windows domain. Each domain has at least one domain controller, usually a Windows NT server providing password

authentication. This domain controller provides the workgroup with a definitive password server. The domain controllers keep track of users and passwords in their own NTDS.dit and Security

Authentication Module (SAM) and authenticate each user when they log in for the first time and wish to access another machine's share.

# RPCclient

We can see from the results that it is not very much that Nmap provided us with here. Therefore, we should resort to other tools that allow us to interact manually with the SMB and send specific

requests for the information. One of the handy tools for this is rpcclient. This is a tool to perform MS-RPC functions.

The Remote Procedure Call (RPC) is a concept and, therefore, also a central tool to realize operational and work-sharing structures in networks and client-server architectures. The communication

process via RPC includes passing parameters and the return of a function value.

```c
rpcclient -U "" 10.129.14.128

Enter WORKGROUP\'s password:
rpcclient $> 

```

```c
Query 	Description
srvinfo 	Server information.
enumdomains 	Enumerate all domains that are deployed in the network.
querydominfo 	Provides domain, server, and user information of deployed domains.
netshareenumall 	Enumerates all available shares.
netsharegetinfo <share> 	Provides information about a specific share.
enumdomusers 	Enumerates all domain users.
queryuser <RID> 	Provides information about a specific user.
```

# Brute Forcing User RIDs
```c
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201
		
        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
		
        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

# Impacket - Samrdump.py
```c
samrdump.py 10.129.14.128

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: 
mrb3n (1000)/UserComment: 
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
mrb3n (1000)/ScriptPath: 
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/UserComment: 
cry0l1t3 (1001)/PrimaryGroupId: 513
cry0l1t3 (1001)/BadPasswordCount: 0
cry0l1t3 (1001)/LogonCount: 0
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
cry0l1t3 (1001)/ScriptPath: 
[*] Received 2 entries.
```

