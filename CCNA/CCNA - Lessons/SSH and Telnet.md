![[Pasted image 20260211174249.png]]![[Pasted image 20260211174404.png]]![[Pasted image 20260211182028.png]]




SSH:

- IOS images that support SSH will have ‘K9’ in their name when using *show version*
- NPE IOS images do not support cryptographic features such as SSH.
- For SSH to work, you must have **FQDN** = Fully Qualified Domain Name (host name + domain name)
**R1 (config)#hostname**
**R1 (config)#ip domain name**
**R1 (config)#crypto key generate rsa**
> enter number of bits in the modulus 360-4096

![[Pasted image 20260218094930.png]]