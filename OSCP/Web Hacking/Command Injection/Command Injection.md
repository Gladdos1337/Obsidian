Command injection is also often known as “Remote Code Execution” (RCE) because of the ability to remotely execute code within an application. These vulnerabilities are often the most lucrative to an attacker because it means that the attacker can directly interact with the vulnerable system. For example, an attacker may read system or user files, data, and things of that nature.

or example, being able to abuse an application to perform the command `whoami` to list what user account the application is running will be an example of command injection.

This vulnerability exists because applications often use functions in programming languages such as PHP, Python and NodeJJS to pass data to and to make system calls on the machine's OS.
For example, taking input from a field and searching for an entry into a file. Take this code snippet:

In this code snippet, the application takes data that a user enters in an input field named `$title` to search a directory for a song title. Let’s break this down into a few simple steps.


![[Pasted image 20260510210258.png]]

**1.** The application stores MP3 files in a directory contained on the operating system.
**2.** The user inputs the song title they wish to search for. The application stores this input into the `$title` variable.
**3.** The data within this `$title` variable is passed to the command `grep` to search a text file named _songtitle.__txt_ for the entry of whatever the user wishes to search for.
**4.** The output of this search of _songtitle.__txt_ will determine whether the application informs the user that the song exists or not.





``ping ; cat /home/tryhackme/flag.txt
``ping ; ls``
``



``


