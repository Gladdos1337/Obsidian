Is a a web security vulnerability allows an attacker to read operating system resources, such as local files on the server running an application. The attacker exploits this vulnerability by manipulating and abusing the web application's URL to locate and access files or directories stored outside the application's root directory.>

Path traversal vulnerabilities occur when the user's input is passed to a function such as **file_get_contents** in PHP. It's important to note that the function is not the main contributor to the vulnerability. Often poor input validation or filtering is the cause of the vulnerability. In , you can use the **file_get_contents** to read the content of a file. 
https://www.php.net/manual/en/function.file-get-contents.php

Windows example:

`http://webapp.thm/get.php?file=../../../../boot.ini`
`http://webapp.thm/get.php?file=../../../../windows/win.ini`

Linux example:

`http://webapp.thm/get.php?file=../../../../etc/passwd`

The same concept applies here as with Linux operating systems, where we climb up directories until it reaches the root directory, which is usually .

Sometimes, developers will add filters to limit access to only certain files or directories. Below are some common OS files you could use when testing.

|**Location**|**Description**|
|`/etc/issue`|contains a message or system identification to be printed before the login prompt.|
|`/etc/profile`|controls system-wide default variables, such as Export variables, File creation mask (umask), Terminal types, Mail messages to indicate when new mail has arrived|
|`/proc/version`|specifies the version of the Linux kernel|
|`etc/passwd`|has all registered users that have access to a system|
|`/etc/shadow`|contains information about the system's users' passwords|
|`/root/.bash_history`|contains the history commands for `root` user|
|`/var/log/dmessage`|contains global system messages, including the messages that are logged during system startup|
|`/var/mail/root`|all emails for `root` user|
|`/root/.ssh/id_rsa`|Private keys for a root or any known valid user on the server|
|`/var/log/apache2/access.log`|the accessed requests for `Apache` web server|
|`C:\boot.ini`|contains the boot options for computers BIOS with firmware
