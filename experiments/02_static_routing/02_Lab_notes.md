Lab notes
1) Inspecting routes (before changes)

After inspecting routes i have following atm

node z3r0
default via 192.168.1.1 dev wlan0 proto dhcp src 192.168.1.179 metric 600

node z3rowh
default via 192.168.1.1 dev wlan0 proto dhcp src 192.168.1.138 metric 600

pi 5
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.181 metric 100
default via 192.168.1.1 dev wlan0 proto dhcp src 192.168.1.183 metric 200


2) Initial setup on rootnode (Pi 5)

rasp pi in default setting behave like a host which has to be adjusted first

enable IP forwarding in the Linux kernel:

sudo sysctl -w net.ipv4.ip_forward=1


3) Create lab network interface (dummy adapter) on rootnode

i have created a fake network adapter called lab0 by using following command

sudo ip link add lab0 type dummy

which looks like following

26: lab0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 5a:71:26:af:72:59 brd ff:ff:ff:ff:ff:ff


after reboot my dummy network was off so i had to recreate it

note to check why

after checking and making sure my dummy network called lab0 is created and is up

26: lab0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether e6:18:13:ac:f7:de brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 scope global lab0
       valid_lft forever preferred_lft forever
    inet6 fe80::e418:13ff:feac:f7de/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever


4) Install Tor on rootnode

after that i installed Tor and checked status

xindy@rootnode:~ $ systemctl status tor
● tor.service - Anonymizing overlay network for TCP (multi-instance-master)
     Loaded: loaded (/usr/lib/systemd/system/tor.service; enabled; preset: enabled)
     Active: active (exited) since Thu 2026-03-05 16:32:28 AEDT; 17s ago
 Invocation: 6a8910fbf90545cf85c863da8f2e0db7
    Process: 1153587 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 1153587 (code=exited, status=0/SUCCESS)
        CPU: 8ms

Mar 05 16:32:28 rootnode systemd[1]: Starting tor.service - Anonymizing overlay network for TCP (multi-instance-master)...
Mar 05 16:32:28 rootnode systemd[1]: Finished tor.service - Anonymizing overlay network for TCP (multi-instance-master).


5) Assign lab IP to Pi Zero (z3r0)

i assigned one of my pi zero in this case pi zero2w hostname z3r0

ive added 10.10.10.2/24

to test it i tried to ping it from rootnode and to see ip a which showed

2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 88:a2:9e:8e:2d:99 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.179/24 brd 192.168.1.255 scope global dynamic noprefixroute wlan0
       valid_lft 78297sec preferred_lft 78297sec
    inet 10.10.10.2/24 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 fe80::8aa2:9eff:fe8e:2d99/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever


6) Ping test (connectivity)

ping from rootnode

xindy@rootnode:~ $ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.051 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.052 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.031 ms
64 bytes from 10.10.10.1: icmp_seq=4 ttl=64 time=0.031 ms
^C
--- 10.10.10.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3067ms
rtt min/avg/max/mdev = 0.031/0.041/0.052/0.010 ms
xindy@rootnode:~ $

Replace with photo
photo name: ping-from-rootnode.png


7) Force traffic to Tor using firewall redirect

now is time to make it force traffic to Tor using firewall redirect with command

sudo iptables -t nat -A PREROUTING -i lab0 -p tcp --syn -j REDIRECT --to-ports 9040

lesson for how this works is in separate file:
lesson_netfilter_prerouting.md


8) Try to route z3r0 -> rootnode -> Tor -> Internet

then adjust ip route on z3r0

z3r0 → rootnode → Tor → Internet


9) Verification issue

in between step when im about to verify problem i had

xindy@z3r0:~ $ curl https://check.torproject.org
curl: (28) Failed to connect to check.torproject.org port 443 after 135415 ms: Could not connect to server
xindy@z3r0:~ $


10) Quick checks on rootnode

after bit of checking on pi 5

xindy@rootnode:~ $ cat /proc/sys/net/ipv4/ip_forward
1

xindy@rootnode:~ $ ip route
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.181 metric 100
default via 192.168.1.1 dev wlan0 proto dhcp src 192.168.1.183 metric 200
10.10.10.0/24 dev lab0 proto kernel scope link src 10.10.10.1
10.244.0.0/16 dev ztyjkmjvmh proto kernel scope link src 10.244.10.4
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.18.0.0/16 dev br-ac539ba2d763 proto kernel scope link src 172.18.0.1
172.19.0.0/16 dev br-80afc3988144 proto kernel scope link src 172.19.0.1
192.168.1.0/24 dev eth0 proto dhcp scope link src 192.168.1.181 metric 100
192.168.1.0/24 dev wlan0 proto dhcp scope link src 192.168.1.183 metric 200

everything seem ok so i assume problem is with z3r0


11) Route cleanup test on z3r0

on pi zero i made it delete a default route

sudo ip route del default via 10.10.10.1

i have to also remove the Tor test IP clean up

tried to test it again and traffic went back to

z3r0 → home router (192.168.1.1) → internet


12) To resolve this i have to (TODO)

to resolve this i have to

- make sure z3r0 is actually using rootnode as default route for the test, and not the home router
- confirm only one default route is active for the test path
- retest curl to check.torproject.org after route changes
- document final working routes and keep a revert command

(when solved, update this section with the final commands + outputs)

![Tor proof](/Photos/torproof.png)

Notes (terms used)

DHCP = Dynamic Host Configuration Protocol
TCP = Transmission Control Protocol
SYN = Synchronize flag (TCP handshake start)
NAT = Network Address Translation
