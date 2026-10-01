# Write-up of the TryHackMe room — Hashing Basics
New week, new room- I present to you the Hashing Basics room.
## Task 1- Introduction
We start by starting the lab machine and logging in with ssh.
## Task 2- Hash Functions
The first question — "_What is the SHA256 hash of the passport.jpg file in ~/Hashing-Basics/Task-2?_" was pretty straight forward. All you need to do is to venture to the ~/Hashing-Basics/Task-2 directory.

Using `ls`, we can make sure the passport.jpg file is in this directory, and running
`sha256sum passport.jpg`
in the directory revealed the SHA256 hash.

--- 
For the question “_What is the output size in bytes of the MD5 hash function?_”- Since MD5 outputs 32 hexadecimal characters, MD5 produces a 16-byte digest. 

We know this, according to the article above, “Remember that hexadecimal format prints each raw byte as two hexadecimal digits.”

You can also confirm this by computing an MD5 hash locally and noting its 32‑hex‑char length. In this case,
`md5sum passport.jpg`
if you want to try it out :)

---
For the third question: “_If you have an 8-bit hash output, how many possible hash values are there?_”

Each bit can be either 0 or 1, so for 8 bits, the total combinations are 2 raised to the power of 8, which equals 256 possible hash values.
## Task 3- Insecure Password Storage for Authentication

The question “_What is the 20th password in rockyou.txt?_”: I would reccomend simply typing

`head -n 20 rockyou.txt`
in the terminal, but if you are not familiar with command line options and prefer another way, you can open rockyou.txt with a text editor, e.g. `nano rockyou.txt`
and manually count.

## Task 4- Using Hashing for Secure Password Storage
We covered rainbow tables, and with that information you can easily find “`inS3CyourP4$$`” for the question “_Manually check the hash “4c5923b6a6fac7b7355f53bfe2b8f8c1” using the rainbow table above._”

---
For the second question, all you need to do is copy and paste the hash on an online hash cracking service. I used [hashes.com](https://hashes.com/en/decrypt/hash).
A problem I had, was not getting the output, which then I realized I didn’t fill in anything in the captcha check (how intelligent of me).

---
For the third question, no we should not encrypt passwords in password-verification systems.
## Task 5- Recognising Password Hashes
This section is very simple- just take notes of the characteristics of each hash. If you have trouble finding the answers-
* What is the hash size in yescrypt? -256
* What’s the Hash-Mode listed for Cisco-ASA MD5? -2410
* What hashing algorithm is used in Cisco-IOS if it starts with $9$? -scrypt

## Task 6- Password Cracking
_Use hashcat to crack the hash, `$2a$06$7yoU3Ng8dHTXphAg913cyO6Bjs3K5lBnwq5FJyA6d01pMSrddr1ZG`, saved in `~/Hashing-Basics/Task-6/hash1.txt._`
For this question, I went to the directory hash1.txt existed, then used the command

`hashcat -m 3200 -a 0 hash1.txt /usr/share/wordlists/rockyou.txt`
* We can know that the hash is bcrypt through recognizing the header, $2a$06$, so the hash type code is 3200.
* -a 0 is the attack mode
Running the command leads to the answer to this question.
<img width="1400" height="831" alt="image" src="https://github.com/user-attachments/assets/50039f99-bf6c-42cb-9e72-4affd1eb83c7" />

---
Use hashcat to crack the SHA2-256 hash, `9eb7ee7f551d2f0ac684981bd1f1e2fa4a37590199636753efe614d4db30e8e1`, saved in `~/Hashing-Basics/Task-6/hash2.txt.`
I did the same thing as the last question, except for replacing hash1.txt with hash2.txt. And there was the cracked password.
<img width="1604" height="1040" alt="image" src="https://github.com/user-attachments/assets/01f2d9f8-5c6e-4101-afbb-54fb82db4ffa" />

---
Same for the third question:
<img width="1400" height="861" alt="image" src="https://github.com/user-attachments/assets/1c253db3-7fe7-41fc-88ac-02a067e6580d" />

---
For the fourth question I used a online cracking service:
[Crackstation](https://crackstation.net).
and the output was “funforyou”.
## Task 7- Hashing for Integrity Checking
What is SHA256 hash of `libgcrypt-1.11.0.tar.bz2` found in `~/Hashing-Basics/Task-7`?

All you need to do is run
`sha256sum libgcrypt-1.11.0.tar.bz2`
in the correct directory, and you will get the SHA256 hash: `09120c9867ce7f2081d6aaa1775386b98c2f2f246135761aae47d81f58685b`

---
What’s the hashcat mode number for HMAC-SHA512 (key = $pass)?
The answer is 1750, just check
[the website for hashcat](https://hashcat.net/wiki/doku.php?id=example_hashes).
## Task 8- Conclusion
Use base64 to decode `RU5jb2RlREVjb2RlCg==`, saved as `decode-this.txt` in `~/Hashing-Basics/Task-8`. What is the original word?

Go to the correct directory, see above and use this command:
`base64 -d decode-this.txt`
And you get the output, ENcodeDEcode.

---
We are done with this room! YAY! You can continue your learning journey with the John the Ripper room, and if you found this helpful please star this respiratory.




