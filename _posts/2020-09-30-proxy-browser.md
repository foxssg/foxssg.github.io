---
published: false
---
## How to proxy traffic through browser

First thing to do on the terminal:
ssh -D 8123 -f -C -q -N root@192.168.1.10
Where 8123 is the port you like and root@192.168.1.10 is the user and machine you like.

Then for example on firefox you have to go to:
preferences
network settings
manual proxy configuration
sock hosts: 192.168.1.10 port: 8123
no proxy for: localhost, 127.0.0.1

You're done, now all the firefox traffic is using that ip and port.


This is great when you want to access to an online service that is only allowed by that ip but you don't want to traffic more things to that ip.