**4.9 Describe the capabilities and function of TFTP/FTP in the network.**

**THE PURPOSE OF FTP / TFTP**
• FTP (==File Transfer Protoco==l) and TFTP (==Trivial File Transfer Protocol==) are INDUSTRY STANDARD
PROTOCOLS used to TRANSFER FILES over a NETWORK
• They BOTH use a CLIENT-SERVER model
o CLIENTS can use FTP / TFTP to COPY files FROM a SERVER
o CLIENTS can use FTP / TFTP to COPY files TO a SERVER
• As a NETWORK ENGINEER, the most common use for FTP / TFTP is in the process of
UPGRADING the OPERATING SYSTEM of a NETWORK DEVICE
• You can use FTP / TFTP to DOWNLOAD the newer version of IOS from a SERVER and then
REBOOT the DEVICE with the new IOS image


**TFTP**
TFTP (1981) is very **SIMPLE**, only has basic features compared to FTP.
- Only allows a CLIENT to COPY FILES to / from a SERVER
Was released after FTP, but not a complete replacement, it has it's place.
- It's very lightweight, so it's good for quick use.
- No authentication so servers will respond to **ALL FTP REQUESTS**
- No encryption, all data is **PLAIN TEXT**.
- Best used in controlled environment to transfer SMALL FILES quickly. 
- ==TFTP SERVERS listen to UDP port 69. (see [[31 - UDP & TCP]])
- UDP is CONNECTIONLESS and doesn't provide **reliability** with **retransitions** 
- However, TFTP has SIMILAR built-in FEATURES within the PROTOCOL itself
TFTP RELIABILITY
* Every TFTP DATA message is **ACKNOWLEDGED**:
	- If the CLIENT is transferring a FILE TO the SERVER, the SERVER will send ACK
	messages
	- If the SERVER is transferring a FILE TO the CLIENT, the CLIENT will send ACK
	messages
	Basically whoever **receives** the files will send ACK message back.
- TIMERS are used, and if an EXPECTED message isn’t received in time, the waiting DEVICE will RESEND its previous message.
![[Pasted image 20251216001808.png]]
![[Pasted image 20251216001833.png]]


**FTP**
- 1971 - before TCP and IP
- Usernames and passwords are used for AUTHENTICATION, however **there is no ENCRPYTION - everything is PLAINTEXT**.
- For greater security, ==**FTPS** (FTP over SSL/TLS)== can be used. - **Upgrade to FTP**
- ==SSH FTP (SFTP)== can also be used for greater security.  - **New Protocol**
- FTP is more complex than TFTP and allows not only file transfers, but:
	- Clients can also navigate file directories
	- Add / Remove directories
	- List files
	- Etc
- The client sends FTP commands to the server to perform these functions.
	https://en.wikipedia.org/wiki/List_of_FTP_commands

**==Uses TCP ports 20 & 21==**:
- **FTP CONTROL** connection (**TCP 21**) is established and ==is used to send FTP commands and replies==.
- **FTP DATA** connection (**TCP 20**) ==is used to transfer data==, and it's established and terminated as needed.
![[Pasted image 20251216002956.png]]
There are 2 types of FTP Data connections:
1. Active mode FTP DATA Connections - (==considered NORMAL mode of initiating FTP connections==):
![[Pasted image 20251216003223.png]]
2. Passive Mode FTP Data Connections - ==when the client is behind a **FIREWALL**.
   ![[Pasted image 20251216003408.png]]

![[Pasted image 20251216003511.png]]

**IOS File Systems** - **WILL PROBABLY NOT BE ON CCNA, TOPIC IS REMOVED.**
- A file system is a way of controlling how data is stored and retrieved.
- You can view the file systems of a Cisco IOS device with:
**Router#show file systems**
![[Pasted image 20251216003737.png]]

How to use TFTP and FTP:
![[Pasted image 20251216003856.png]]
**R1#show version** - view current version of IOS
![[Pasted image 20251216004005.png]]
**R1#show flash** - view the contents of flash
![[Pasted image 20251216004029.png]]
What we're going to do now is use TFTP to copy new the new version of IOS from SRV1 to R1, configure R1 to boot with new IOS and then delete old one from flash.

**R1#copy tftp: flash:** - copy *source* *destination*
Router then asks for several questions:
**Address or name of remote host**: *Enter the IP address of the TFTP server.*
**Source filename []?** - *Enter the name on the server.*
**Destination filename** - *Enter the name of the file you're saving, **just hit enter to use the default name**.*

**R1#show flash**
*We will get a number of the new IOS file*

**R1(config)#boot system flash:** ***(flash file name)***
If you don't use this command, the router will use the first IOS file it finds in flash, we need to force it to use the new version.

**R1#write memory** - duh

**R1#reload** - restart the device

After restart...

**R1#show version**

**R1#delete flash: (flash file name)** - deleting the old one.

FTP (just a bit):

![[Pasted image 20251216005215.png]]

![[Pasted image 20251216005139.png]]
All commands needed:

![[Pasted image 20251216005228.png]]
