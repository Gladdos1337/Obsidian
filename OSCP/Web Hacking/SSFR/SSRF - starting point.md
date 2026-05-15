Related: [[Defeating Common SSRF Defenses]] | [[File Inclusion]] | [[Insecure direct object references (IDOR)]]

SSRF stands for  Server-Side Request Forgery. It's a vulnerability that allows a malicious user to cause the webserver to make an additional or edited HTTP request to the resource of the attacker's choosing.

There are 2 types of SSRF vulns

1. regular SSRF where data is returned to the attacker's screen
2. blind SSRF where an SSRF occurs, but no info is reutrned to attacker

A successful SSRF attack can result in any of the following:

- Access to customer/organisational data.
- Ability to Scale to internal networks.
- Reveal authentication tokens/credentials.
- Access to unauthorised areas.

**Task: Using what you've learnt, try changing the address in the browser below to force the webserver to return data from https://server.website.thm/flag?id=9**. To make things easier the Server Requesting bar at the bottom of the mock browser will show the URL that website.thm is requesting.**

**https://website.thm/item/2?server=api**
**to**
**https://website.thm/item/2?server=server.website.thm/flag?id=9&x=**

![[Pasted image 20260509142358.png]]

![[Pasted image 20260509142419.png]]
![[Pasted image 20260509142426.png]]![[Pasted image 20260509142435.png]]![[Pasted image 20260509142445.png]]