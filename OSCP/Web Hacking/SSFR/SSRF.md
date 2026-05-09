SSRF stands for  Server-Side Request Forgery. It's a vulnerability that allows a malicious user to cause the webserver to make an additional or edited HTTP request to the resource of the attacker's choosing.

There are 2 types of SSRF vulns

1. regular SSRF where data is returned to the attacker's screen
2. blind SSRF where an SSRF occurs, but no info is reutrned to attacker

A successful SSRF attack can result in any of the following:

- Access to customer/organisational data.
- Ability to Scale to internal networks.
- Reveal authentication tokens/credentials.
- Access to unauthorised areas.