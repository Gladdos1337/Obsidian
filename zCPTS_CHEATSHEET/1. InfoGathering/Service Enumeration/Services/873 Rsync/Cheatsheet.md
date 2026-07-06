```
#Scanning for Rsync
sudo nmap -sV -p 873 127.0.0.1

#Probing for Accessible Shares
nc -nv 127.0.0.1 873

#Enumerating an Open Share
rsync -av --list-only rsync://127.0.0.1/dev

From the above output, we can see a few interesting files that may be worth pulling down to investigate further. We can also see that a directory likely containing SSH keys is accessible. From here, we could sync all files to our attack host with the command rsync -av rsync://127.0.0.1/dev. If Rsync is configured to use SSH to transfer files, we could modify our commands to include the -e ssh flag, or -e "ssh -p2222" if a non-standard port is in use for SSH. This guide is helpful for understanding the syntax for using Rsync over SSH.
```


[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files. It can be used to copy files locally on a given machine and to/from remote hosts. It is highly versatile and well-known for its delta-transfer algorithm. This algorithm reduces the amount of data transmitted over the network when a version of the file already exists on the destination host. It does this by sending only the differences between the source files and the older version of the files that reside on the destination server. It is often used for backups and mirroring. It finds files that need to be transferred by looking at files that have changed in size or the last modified time. By default, it uses port `873` and can be configured to use SSH for secure file transfers by piggybacking on top of an established SSH server connection.

Abusing Rsync

https://hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html