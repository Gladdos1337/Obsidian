```
#Footprinting the service
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

# Banner grabbing / manual interaction 
nc -nv <target-ip> 3306

# Targeted Nmap auxiliary scripts for MySQL 
sudo nmap -p3306 --script mysql-empty-password,mysql-brute,mysql-users,mysql-enum,mysql-variables <target-ip>



```



MySQL is ideally suited for applications such as `dynamic websites`, where efficient syntax and high response speed are essential. It is often combined with a Linux OS, PHP, and an Apache web server and is also known in this combination as [LAMP](https://en.wikipedia.org/wiki/LAMP_\(software_bundle\)) (Linux, Apache, MySQL, PHP), or when using Nginx, as [LEMP](https://lemp.io/). In a web hosting with MySQL database, this serves as a central instance in which content required by PHP scripts is stored. Among these are:

|                         |                  |                   |            |
| ----------------------- | ---------------- | ----------------- | ---------- |
| Headers                 | Texts            | Meta tags         | Forms      |
| Customers               | Usernames        | Administrators    | Moderators |
| Email addresses         | User information | Permissions       | Passwords  |
| External/Internal links | Links to Files   | Specific contents | Values     |
Sensitive data such as passwords can be stored in their plain-text form by MySQL; however, they are generally encrypted beforehand by the PHP scripts using secure methods such as [One-Way-Encryption](https://en.citizendium.org/wiki/One-way_encryption).