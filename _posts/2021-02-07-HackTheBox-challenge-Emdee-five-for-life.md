---
published: false
---
## Emdee five for life

Can you encrypt fast enough?

Actually, 'encrypt' is the wrong choice of wording because MD5 is a hashing algorithm, not an encryption algorithm. Anyway...

The site gives you a string to MD5 hash, but if you do it manually it will time out before you can get the flag. So some scripting skill is required.

This is what I used to get the flag:

```python
#!/usr/bin/env python3

from bs4 import BeautifulSoup
import requests
import hashlib

url = 'http://134.209.16.184:31490/'
r = requests.session()
page = r.get(url)
soup = BeautifulSoup(page.text, 'lxml')
to_encrypt = soup.find('h3').text
result = hashlib.md5(to_encrypt.encode('utf-8')).hexdigest()
data = {'hash': result}
sendit = r.post(url = url, data = data)
print(sendit.text)
```
