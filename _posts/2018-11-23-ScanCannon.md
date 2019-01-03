---
published: true
comments: true
---
[ScanCannon](https://github.com/sdcampbell/ScanCannon) - Runs Masscan, followed up by Nmap for service version info. This should finish scans much faster than Nmap alone while providing service version info that Masscan doesn't provide.

During testing, a scan using nmap alone against all 65535 ports on 7 hosts on a /24 took more than two hours to reach 70 percent. Using ScanCannon, the same nmap parameters finished in 31 minutes.

Note that the largest time savings from using ScanCannon will be seen when scanning all TCP ports. You won't see any time savings if you scan only a limited number of ports.
