Related: [[(THM) EasyCTF]] | [[OSCP/Boxes/THM/(THM) EasyCTF/1enum]] | [[Brute force]]

21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT


Connected to 10.112.138.49.
220 (vsFTPd 3.0.3)
Name (10.112.138.49:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> passive
Passive mode: off; fallback to active mode: off.