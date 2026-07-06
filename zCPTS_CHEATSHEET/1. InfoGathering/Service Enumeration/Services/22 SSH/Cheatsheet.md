```
#SSH-Audit - Footprinting
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132

-For potential brute-force attacks, we can specify the authentication method with the SSH client option PreferredAuthentications

ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
```






In a first step, the SSH server and client authenticate themselves to each other. The server sends its `public host key` to the client, which the client uses to verify the server's identity. Only when contact is first established there is a risk of a third party interposing itself between the two participants and thus intercepting the connection. A `host key` cannot be imitated because it is a unique public-private `key pair`, and an attacker cannot forge the private key’s signature without access to it, assuming the client properly verifies the public key against a trusted source.

After server authentication, however, the client must also prove to the server that it has access authorization. However, the SSH server is already in possession of the encrypted hash value of the password set for the desired user. As a result, users have to enter the password every time they log on to another server during the same session. For this reason, an alternative option for client-side authentication is the use of a public key and private key pair.