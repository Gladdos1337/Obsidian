Related: [[Tools]] | [[Types of shells]] | [[Socat]] | [[rlwrap]] | [[RFI (Remote File Inclusion)]]

The first technique we'll be discussing is applicable only to **LINUX** boxes, as they will nearly always have Python installed by default. This is a three stage process:


The first technique we'll be discussing is applicable only to

boxes, as they will nearly always have Python installed by default. This is a three stage process:

1. The first thing to do is use `python -c 'import pty;pty.spawn("/bin/bash")'`, which uses Python to spawn a better featured bash shell; note that some targets may need the version of Python specified. If this is the case, replace `python` with `python2` or `python3` as required. At this point our shell will look a bit prettier, but we still won't be able to use tab autocomplete or the arrow keys, and Ctrl + C will still kill the shell.
2. Step two is: `export TERM=xterm` -- this will give us access to term commands such as `clear`.
3. Finally (and most importantly) we will background the shell using Ctrl + Z. Back in our own terminal we use `stty raw -echo; fg`. This does two things: first, it turns off our own terminal echo (which gives us access to tab autocompletes, the arrow keys, and Ctrl + C to kill processes). It then foregrounds the shell, thus completing the process.
![[Pasted image 20260512165652.png]]

Note that if the shell dies, any input in your own terminal will not be visible (as a result of having disabled terminal echo). To fix this, type `reset` and press enter.



python -c 'import pty;pty.spawn("/bin/bash")'
CTRL + Z
export TERM=xterm
stty raw -echo; fg
reset