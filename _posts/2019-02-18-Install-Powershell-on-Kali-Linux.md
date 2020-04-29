---
published: true
---
Update: This post is outdated. The Kali repo's are fixed and now the best way to install PowerShell on Kali is to run this command: `apt update && apt -y install powershell`

I tried to install PowerShell on Kali Linux Rolling by following instructions on the GitHub page as well as other articles I found online and none of them worked. I'm going to tell you what worked for me.

In the past I've stuck to Bash and Python for all of my scripting needs because they work cross platform. My work issued laptop runs Windows 10 and I use Git Bash to run my simple shell scripts that I use mainly to slice, dice, and reformat data, and Python for everything else. I'm a big fan of using one cross platform scripting language when possible.

Lately I've found a need to dive into PowerShell to be able to understand a complex script that I took over from a departing coworker. I was really surprised at how easy it is to work with XML using PowerShell after struggling to read XML with Python and xmlstarlet. Add in some Unicode and dependency problems while switching back and forth between Python 2.7 and 3.5 and I knew is was time to give PowerShell a chance. This had me thinking about starting a personal project to create a cross platform script in PowerShell to manage pentests and reporting.

Let's get started installing PowerShell on Kali.

```
apt update && apt -y install curl gnupg apt-transport-https
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-stretch-prod stretch main" > /etc/apt/sources.list.d/powershell.list
apt update
wget http://http.us.debian.org/debian/pool/main/i/icu/libicu57_57.1-6+deb9u2_amd64.deb
dpkg -i libicu57_57.1-6+deb9u2_amd64.deb
apt -y install powershell
```

Now start PowerShell:
  
`pwsh`

If you get any errors when using certain PowerShell commands, like curl for example, check your aliases. Some common aliases that work by default on Windows aren't set here. You'll need to either use the expanded name or set a new alias.
