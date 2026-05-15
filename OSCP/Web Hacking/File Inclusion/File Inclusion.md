Related: [[LFI (Local File Inclusion)]] | [[RFI (Remote File Inclusion)]] | [[Directory or Path Traversal]] | [[Command Injection]] | [[SSRF - starting point]]

![[Pasted image 20260507143551.png]]

Local File Inclusion (**LFI**) → [[LFI (Local File Inclusion)]]
Remote File Inclusion (**RFI**) → [[RFI (Remote File Inclusion)]]
Directory traversal → [[Directory or Path Traversal]]

Let's discuss a scenario where a user requests to access files from a webserver. First, the user sends an HTTP request to the webserver that includes a file to display. For example, if a user wants to access and display their CV within the web application, the request may look as follows, http://webapp./get.?file=userCV.pdf, where the **file** is the parameter and the **userCV.pdf**, is the required file to access.

When the input is not validated, the user can pass any input to the function, causing the vulnerability.


