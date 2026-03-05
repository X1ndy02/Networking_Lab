Connectivity Validation

Date: 2026-03-05

Purpose
Make all devices conectable outside of lan adn Verify that all nodes in the lab can communicate over the ZeroTier virtual network after the initial hardware build and network setup

Nodes tested
- Pi 5 (rootnode)
- Pi Zero2W (z3r0)
- Pi Zero (p1z3r0wh)
- Mac (management machine)

![Hostnames](experiments/01_connectivity_validation/Hostnames.png)

Test method

ICMP connectivity test was performed using the ping command on Headless Pi5

Commands used + results 
![Ping Command](experiments/01_connectivity_validation/Ping_Command.png)

Observations

All nodes responded successfully with no packet loss.
Latency varies depending on node but remains within expected limits for a ZeroTier network

Conclusion

Connectivity across the ZeroTier network is functioning correctly.
This confirms that the lab environment is ready for further networking experiments such as routing, firewall testing, and packet inspection.