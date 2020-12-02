If you want to acces some port (for example to access a web app) but you cannot acces directly to that machine you can do:


ssh -L 0.0.0.0:8124:10.120.59.1:8001 root@192.168.1.1

you can change 8124 to any number you like (that is not being used)
10.120.59.1 is the host machine
8001 is the port in the host that you want to access
root@192.168.1.1 is the machine that has access to the host machine

In this example 192.168.1.7 is the intermediary machine (in which we ran the previous ssh command)
Now in our local machine in the browser we can go to: 192.168.1.7:8124 and this is going to redirect to 10.120.59.1:8001


