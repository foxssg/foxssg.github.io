---
layout: post
title: Linux-Namespaces
published: true
---
I'm sharing a little trick I've learned recently, from this video https://www.youtube.com/watch?v=d2KnVasIq5I
I'm using when I need to use a client vpn to run a bash so I can use ssh, and run others vpn at the same time.



# Running specific applications through a VPN with network namespaces

# Initial state
curl ifconfig.co/city
sudo ip link list
sudo ip netns list

# Disable Uncomplicated Firewall (Ubuntu)
sudo ufw disable

# Create network namespace - myvpn
sudo ip netns add myvpn

# List existing network namespaces
ip netns list

# Add loopback adapter to myvpn and start networking
sudo ip netns exec myvpn ip addr add 127.0.0.1/8 dev lo
sudo ip netns exec myvpn ip link set lo up

# Create a veth pair - vpndest0 and vpnsource0
# (This will be our tunnel from vpnsource0 (inside myvpn) to vpndest0)
sudo ip link add vpndest0 type veth peer name vpnsource0
sudo ip link set vpndest0 up
sudo ip link set vpnsource0 netns myvpn up

# List existing network interfaces
sudo ip link list

# Assign local IP address to vpnsource0 and vpndest0
# Route all traffic inside vpnsource0 to vpndest0
sudo ip addr add 10.200.200.1/24 dev vpndest0
sudo ip netns exec myvpn ip addr add 10.200.200.2/24 dev vpnsource0
sudo ip netns exec myvpn ip route add default via 10.200.200.1 dev vpnsource0

# Configure iptables
# Route all traffic from vpndest0 to ethernet - e+ to match enp...
# Change to -o w+ if using WiFi
iptables -t nat -A POSTROUTING -s 10.200.200.0/24 -o e+ -j MASQUERADE

# Allow ipv4 forwarding
sysctl -q net.ipv4.ip_forward=1

# Use Google DNS server for both inside and outside VPN
# (avoids issues if ISP DNS is private)
sudo mkdir -p /etc/netns/myvpn
sudo sh -c "echo 'nameserver 8.8.8.8' > /etc/netns/myvpn/resolv.conf"
sudo sh -c "echo 'nameserver 8.8.8.8' > /etc/resolv.conf"

# Start OpenVPN inside myvpn network namespace
sudo ip netns exec myvpn openvpn --config /etc/openvpn/client.conf &

# Start firefox inside myvpn network namespace as archie
# We specify the user so we keep our profile, etc.
sudo ip netns exec myvpn sudo -u archie curl ifconfig.co/city
sudo ip netns exec myvpn sudo -u archie firefox


# Undo previous steps and shutdown
sudo sysctl -q net.ipv4.ip_forward=0
sudo iptables -t nat -D POSTROUTING -s 10.200.200.0/24 -o e+ -j MASQUERADE

sudo killall openvpn
sudo ip link delete vpndest0
sudo ip netns delete myvpn
