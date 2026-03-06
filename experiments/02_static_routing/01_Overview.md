Goal

the goal is to test static routing
make all pi haveing static IP base on my choosing
and route traffic via Tor as a experiment


Topology shoudl look like this:

z3r0 / p1z3r0wh
        │
        │ 10.10.10.0/24
        │
     rootnode
        │
      TOR
        │
     Internet


What i did so far

- inspected current routes on all devices (baseline)
- enabled IP forwarding on rootnode so it can act like a router
- created a lab network interface (lab0) and assigned 10.10.10.1/24
- assigned z3r0 an additional static IP 10.10.10.2/24
- confirmed connectivity with ping
- installed Tor on rootnode
- added firewall redirect rule to push TCP traffic into Tor (port 9040)
- tested from z3r0 with curl, but connection failed, so routing path is not fully working yet


Devices

rootnode = Raspberry Pi 5
z3r0 = Pi Zero 2W
p1z3r0wh = Pi Zero
