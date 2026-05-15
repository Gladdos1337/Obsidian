
Related: [[Brute force]] | [[File Inclusion]] | [[SSRF - starting point]]

IDOR stands for Insecure Direct Object Reference and is a type of access control vulnerability.  

This type of vulnerability can occur when a web server receives user-supplied input to retrieve objects (files, data, documents), too much trust has been placed on the input data, and it is not validated on the server-side to confirm the requested object belongs to the user requesting it.

If you can view the other users' content using their Id number while still being logged in with a different account (or not logged in at all), you've found a valid **IDOR** vulnerability.

**Where are they located**

The vulnerable endpoint you're targeting may not always be something you see in the address bar. It could be content your browser loads in via an AJAX request or something that you find referenced in a JavaScript file.

Sometimes endpoints could have an unreferenced parameter that may have been of some use during development and got pushed to production. For example, you may notice a call to **/user/details** displaying your user information (authenticated through your session). But through an attack known as **parameter mining**, you discover a parameter called **user_id** that you can use to display other users' information, for example, **/user/details?user_id=123**.
