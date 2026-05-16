




---
prio list:
80 http ->
http://10.112.173.51/admin/ -> Password probably encrypted with their program, probably not bruteforcable. ??

Ninja
Pars
Szymex
Bee
MuirlandOracle


22 ssh ->  

Program itself -> uses ROT47

---

22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 03:64:6a:b3:33:47:d4:cd:a1:d9:03:d9:bb:bb:71:41 (RSA)
|   256 7d:12:51:b5:a8:fe:64:c1:09:e4:11:c1:67:7d:34:51 (ECDSA)
|_  256 3c:a2:10:8f:08:7f:41:15:8d:52:60:a8:70:6c:6a:36 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass




