# Tryhackme Introduction to SIEM walkthrough with answers

Hello. I’m almost 80% through the whole cybersecurity 101 course, so you might receive some more defensive security walkthroughs for a while. Today I bring to you Introduction to SIEM.

## Task 1 Introduction

_What does SIEM stand for?_


**Security Information and Event Management system**

(Basic stuff, just read the material)

## Task 2 Logs everywhere, Answers nowhere

Interesting name…

So we don’t see logs because of numerous log sources, no centralisation, limited context, limited analysis and format issues.

_Is Registry-related activity host-centric or network-centric?_

**host-centric**

---

_Is VPN-related activity host-centric or network-centric?_

**network-centric**

## Task 3 Why SIEM?

Because it centralises log collection and normalises/correlates logs. Also, SIEM includes real-time alerting, dashboards and reporting.

## Task 4 Log sources and Ingestion

We looked at different types of devices, e.g. windows, linux… and learned where the logs are stored.

Web servers, log ingestion… Quite self-explanatory, again.

_In which location within a Linux environment are HTTP logs stored?

**/var/log/httpd**

## Task 5 Alerting process and analysis

What happens behind the alerts, detection rules and alert investigation.

_Which Event ID is generated when event logs are removed?_

**104**

---

_What type of alert may require tuning?_

**false positive**

## Task 6 Lab Work


<img width="1507" height="1568" alt="image" src="https://github.com/user-attachments/assets/bafab607-cde9-45ec-8dd9-de17f213b70e" />


_After clicking on the Start Suspicious Activity button, which process caused the alert?_

**cudominer.exe**

---

_Find the event that caused the alert and identify the user responsible for the process execution._


<img width="1484" height="1551" alt="image" src="https://github.com/user-attachments/assets/3460ef45-5e2f-4146-9e66-2ca2d829fc71" />

**Chris**

He executed cudominer.exe.

---

_What is the hostname of the suspect user?_

**HR_02**

(You can see in the picture above.)

_Examine the rule and the suspicious process; which term matched the rule that caused the alert?


<img width="1400" height="1389" alt="image" src="https://github.com/user-attachments/assets/2abfae76-b779-430d-bef1-ef4200ae76b7" />

---

_After clicking on the alert, we see the event:_

**miner**

---

_Which option best represents the event? Choose from the following:_

_-False Positive_

_-True Positive_

**True Positive**

---

_Selecting the right ACTION will display the FLAG. What is the FLAG?_


<img width="1400" height="1453" alt="image" src="https://github.com/user-attachments/assets/37e0803c-7c97-4abf-97ff-4045148849e9" />

<img width="1400" height="890" alt="image" src="https://github.com/user-attachments/assets/c7cb5256-916e-4bd3-81ad-aebb00d1e74e" />

**THM{000_SIEM_INTRO}**

## Task 8 Conclusion

Well, we covered SIEM and that’s all for this week! If you found this helpful please leave a like/comment or subscribe.

BTW I’m on LinkedIn now: www.linkedin.com/in/gabriella-kwok-8964a441b

Bye!
