---
published: true
comments: true
---
Kali Linux recently switched from the Bash shell to Zsh. I log the output of every command run during a pentest to a logfile, in addition to saving screenshots. Pentesters are frequently asked by the client or Blue Team for information to correlate with SIEM alerts, so it's a good idea to update your Zsh prompt to include the date, time, and IP address. 

Zsh has the option of adding an 'RPROMPT', or right side prompt. We'll use this to avoid cluttering up the normal prompt on the left. In Kali, edit .zshrc and comment out the line that begins with 'RPROMPT' add the following line below it:

![2020-12-22_09-31.png]({{site.baseurl}}/images/2020-12-22_09-31.png)

After saving your changes, to update the prompt you can either close and relaunch the prompt, or update it immediately by entering `source .zshrc`

Now when you log all cli output to a logfile using `script mylogfile.log`, the date/time and your IP address will be displayed on the right side as shown below:

![2020-12-22_09-08.png]({{site.baseurl}}/images/2020-12-22_09-08.png)

If you're performing and external network penetration test, change the part of the prompt that begins with 'ifconfig' to `curl http://ipecho.net/plain; echo`, but be aware that any network latency may cause a slight delay before the prompt is displayed each time you press enter.
