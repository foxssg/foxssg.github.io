---
published: false
---
## How to proxy traffic through browser

First thing is doing:
ssh -D

On the terminal, then for example on firefox you have to go to:
preferences
network settings
manual proxy configuration
sock hosts: ip
no proxy for: ip

You're done, now all the firefox traffic is using that ip and port.