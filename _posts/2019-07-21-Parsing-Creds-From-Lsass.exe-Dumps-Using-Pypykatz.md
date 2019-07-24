---
published: true
comments: true
---
I rarely use Mimikatz for more than parsing memory dumps of lsass.exe taken with procdump64.exe. I'm just not going to risk running Mimikatz from CrackMapExec or uploading Mimikatz to the client's environment when I can fly under the radar by using wmiexec.py from Impacket to upload procdump64.exe, run the command to make a dump file from lsass.exe, and download it to be processed offline using Mimikatz on a system that I control. If this sounds like a lot of extra steps, it is. This post is about using a Python3 library to save yourself some work when processing those lsass.exe dump file to get credentials.

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

Write a script to include Impacket and run the commands from start to finish and dump creds and grep credentials from the output. Stay tuned for an upcoming script that I have in the works that will do all of the above in one go.
