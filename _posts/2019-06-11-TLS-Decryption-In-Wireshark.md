---
published: true
title: TLS Decryption in Wireshark
---
While testing web applications, I monitor the application using Wireshark to see if the app is using a protocol that lacks support in Burp Suite, like HTTP2. This post shows how to decrypt TLS traffic in Wireshark on Kali Linux.

## Edit .bashrc
If you want to make this permanent, add the following line to your .bashrc file. Otherwise, enter it in the same terminal before starting Chromium from the terminal in a later step.
`export SSLKEYLOGFILE=/root/ssl-key.log`

## Browser notes
Kali Linux is a Debian derivative, which doesn't [enable](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Key_Log_Format) support for an SSLKEYLOGFILE file in Firefox at compile-time. If you don't want to compile Firefox yourself, you'll need to install Chromium.

After adding the export directive to your .bashrc file, source it using `source .bashrc` then start Chromium. Alternatively you may enter the command in your terminal instead of editing .bashrc. If you're running as root on Kali then you'll need to start Chromium using the cli `--no-sandbox` flag.

## Enabling the SSLKEYLOGFILE in Wireshark
In Wireshark, select Edit > Preferences > Protocols > SSL. In the '(Pre)-Master-Secret log filename' field, enter the path to your file. Check the two boxes that start with 'Reassemble SSL...'. Save, then close and reopen Wireshark.

After opening Chromium, browse to a few TLS enabled sites and then check for the precense of your ssl-key.log file. Go back to Wireshark and enter 'ssl' in the filter. When you select packets with TLS DATA, in the lower pane you should see a new tab for 'Decrypted SSL'. Keep in mind that if the decrypted data still looks like garbage, it is likely gzipped.

## Bonus: Curl suports a SSLKEYLOGFILE too!
Curl also supports SSLKEYLOGFILE and any TLS traffic generated with curl can be decrypted by Wireshark.
