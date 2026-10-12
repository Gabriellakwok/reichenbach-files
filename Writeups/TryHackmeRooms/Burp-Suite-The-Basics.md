# TryHackMe- Burp Suite: The Basics Walkthrough
Hello everybody, this is the room for this week.
## Task 1 Introduction
We “aim to understand the basics of the Burp Suite web application security testing framework” in this room. Let’s start.
## Task 2 What is BurpSuite
Read the provided information. At its core, Burp Suite is a tool that helps us capture and manipulate all the HTTP/HTTPS traffic between a browser and a web server.

Upon reading a brief introduction of the different editions that BurpSuite offers, we answer the first question:

_Which edition of Burp Suite runs on a server and provides constant scanning for target web apps?_

**Burp Suite Enterprise**

---

_Burp Suite is frequently used when attacking web applications and ______ applications._

**Mobile**

## Task 3 Features of Burp Community

A outline of the features: Proxy, Repeater, Intruder, Decoder, Comparer, Sequencer.

_Which Burp Suite feature allows us to intercept requests between ourselves and the target?_

**Proxy**         

_Which Burp tool would we use to brute-force a login form?_

**Intruder**

## Task 4 Installation

If you are using the attack box, you can skip this part. If you want to download it, I would recommend downloading it directly from the site: [here](https://portswigger.net/burp/releases)

## Task 5 The Dashboard
Well, the dashboard consists of:

**Tasks**: define background tasks that Burp Suite will perform while you use the application. The default “Live Passive Crawl” task, is sufficient for our purposes in this module.

**Event log**: provides information about the actions performed by Burp Suite, as well as details about connections made through Burp.

**Issue Activity**: This section is specific to Burp Suite Professional. It displays the vulnerabilities identified by the automated scanner, ranked by severity and filterable based on the certainty of the vulnerability.

**Advisory**: The Advisory section provides more detailed information about the identified vulnerabilities, including references and suggested remediations. In Burp Suite Community, this section may not show any vulnerabilities.

_What menu provides information about the actions performed by Burp Suite, such as starting the proxy, and details about connections made through Burp?_

**Event log**

## Task 6 Navigation

Module Selection, Sub-Tabs, Detaching Tabs, and keyboard shortcuts.

_Which tab Ctrl + Shift + P will switch us to?_

**Proxy tab**

## Task 7 Options

Basically we’re talking about settings here.

_In which category can you find a reference to a “Cookie jar”?_

**Sessions**

---

_In which base category can you find the “Updates” sub-category, which controls the Burp Suite update behaviour?_

**Suite**

---

_What is the name of the sub-category which allows you to change the keybindings for shortcuts in Burp Suite?_

**Hotkeys**

---

_If we have uploaded Client-Side TLS certificates, can we override these on a per-project basis (yea/nay)?_

**yea**

## Task 8 Introduction to the Burp Proxy

Basically some things we need to understand about the Burp Proxy: intercepting requests, taking control capture and logging WebSocket support logs and history. Also some proxy options.

## Task 9 Connecting through the Proxy (FoxyProxy)

-> For lab machine users…

Start the lab machine. Open Burp Suite Community Edition, and start a new project.

Open Firefox, and then the extension FoxyProxy.

<img width="1351" height="1410" alt="image" src="https://github.com/user-attachments/assets/d442c571-9e63-4c5e-82f5-ccb0967ac2d3" />

<img width="1384" height="1447" alt="image" src="https://github.com/user-attachments/assets/6dc2dde2-bc6a-404b-b679-a58d38bc6e81" />

Follow everything here, and activate the configuration.

Turn on intercept in our Burp Suite tab.

Then let’s try it:

In firefox, type the url
 `http://Lab_machine_IP/`
 
This was actually my first time trying Burp Suite, so I was confused when the website didn’t seem to load. Apparently all you need to do is press the “forward” button…

<img width="1259" height="605" alt="image" src="https://github.com/user-attachments/assets/fc7c3e35-766a-4a37-99de-6445dfd6f3b8" />

## Task 10 Site Map and Issue definitions

Continuing task 9, we press “Target” and we see this

<img width="1400" height="1324" alt="image" src="https://github.com/user-attachments/assets/b771ee9b-8f71-4013-b797-1b96760c08ec" />

The request that is shown in the picture above, I find suspicious. So I double clicked the request, and pressed “copy URL”. After that, we open our firefox tab, paste it in, and there we are!

Our answer is: **THM{NmNlZTliNGE1MWU1ZTQzMzgzNmFiNWVk}**

## Task 11 The Burp suite browser

After fumbling with FoxyProxy, we realize we can simply open a browser inside burp suite!!!! Yay!!! WOW!!! (rest assured this is sarcasm)

Before using this, configure the settings- just read the materal provided, no questions shall be answered.

## Task 12 Scoping and Targeting

Add `http://10.49.129.39/` to your scope and change the proxy settings to only intercept traffic to in-scope targets.

_See the difference between the amount of traffic getting caught by the proxy before and after limiting the scope._

This is quite simple. Just read the instructions provided and you’ll be fine.

## Task 13 Proxying HTTPS
We don’t need to do anything for the attack box, and if you want to, follow the instructions and set up the certificate and all.

## Task 14 Example Attack

Follow the instructions, and we have a successful attack. Obviously this wouldn’t work in a well-protected environment, but understanding this task is critical as a foundation for further exploiting vulnerablities.

## Task 15 Conclusion
Well, this was fun. As the last question says, “I understand the fundamentals of using Burp Suite!”. Looking forward to seeing u in the next walkthrough! If you found this helpful please star this respo!

