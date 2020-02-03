---
published: false
---
Ruby-Nessus is a ruby interface for the popular Nessus vulnerability scanner. Ruby-Nessus aims to deliver an easy yet powerful interface for interacting and manipulating Nessus scan results and configurations. Ruby-Nessus currently supports both version 1.0 and 2.0 of the .nessus file format.

If you install from rubygems.org using the command `gem install ruby-nessus`, you're going to get version 1.2. If you want the latest version (2.0 beta), install from Github using the following commands:

```
git clone https://github.com/mephux/ruby-nessus
cd ruby-nessus/
gem build ruby-nessus.gemspec
gem install ruby-nessus-2.0.beta.gem
```

Now let's play around with it a bit. In this example I'm going to search the Nessus report for any hosts with vulnerabilities which can be exploited by Metasploit.

```ruby
require 'ruby-nessus'

ness = RubyNessus::Parse.new("/path/to/nessus_report.nessus")

ness.scan.each_host do |host|
  host.each_event do |event|
    if event.exploit_framework_metasploit
      puts "#{host.ip}\t#{event.name}"
    end
  end
end
```

Output:

```
172.17.1.96	MS08-067: Microsoft Windows Server Service Crafted RPC Request Handling Remote Code Execution (958644) (ECLIPSEDWING) (uncredentialed check)
172.17.1.96	MS17-010: Security Update for Microsoft Windows SMB Server (4013389) (ETERNALBLUE) (ETERNALCHAMPION) (ETERNALROMANCE) (ETERNALSYNERGY) (WannaCry) (EternalRocks) (Petya) (uncredentialed check)
172.17.1.96	MS09-001: Microsoft Windows SMB Vulnerabilities Remote Code Execution (958687) (uncredentialed check)
172.17.1.93	MS08-067: Microsoft Windows Server Service Crafted RPC Request Handling Remote Code Execution (958644) (ECLIPSEDWING) (uncredentialed check)
172.17.1.93	MS17-010: Security Update for Microsoft Windows SMB Server (4013389) (ETERNALBLUE) (ETERNALCHAMPION) (ETERNALROMANCE) (ETERNALSYNERGY) (WannaCry) (EternalRocks) (Petya) (uncredentialed check)
172.17.1.93	MS12-020: Vulnerabilities in Remote Desktop Could Allow Remote Code Execution (2671387) (uncredentialed check)
172.17.1.50	Microsoft RDP RCE (CVE-2019-0708) (BlueKeep) (uncredentialed check)
```

Read more at https://github.com/mephux/ruby-nessus.

