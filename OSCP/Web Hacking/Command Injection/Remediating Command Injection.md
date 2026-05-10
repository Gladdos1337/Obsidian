CI an be prevented in a variety of ways. Everything from minimal use of potentially dangerous functions or libraries in a programming language to filtering input without relying on a user’s input. I have detailed these a bit further below. The examples below are of the PHP rogramming language; however, the same principles can be extended to many other languages.

**Vulnerable Functions**

In PHP, many functions interact with the operating system to execute commands via shell; these include:

- Exec
- Passthru
- System

Take this snippet below as an example. Here, the application will only accept and process numbers that are inputted into the form. This means that any commands such as `whoami` will not be processed.!

```
<input type"text" id="ping" name="ping" pattern="[0-9]+"></input>
<?php
echo passthru("/bin/ping -c 4 "$_GET["ping"]-');
?>
```

1. The application will only accept a specific pattern of characters (the digits  0-9)
2. The application will then only proceed to execute this data which is all numerical.

These functions take input such as a string or user data and will execute whatever is provided on the system. Any application that uses these functions without proper checks will be vulnerable to command injection.