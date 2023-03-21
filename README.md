# Lab Setup

![Picture of current lab setup](Lab%20v2%20Layout.png)

[[Hardware Setup]](https://github.com/EddyIAM/Lab/blob/main/Hardware%20setup.md)
[[Linux NUC Setup]](https://github.com/EddyIAM/Lab/blob/main/Linux%20NUC%20setup.md)
[[Windows Setup]](https://github.com/EddyIAM/Lab/blob/main/Windows%20setup.md)
[[Docker Setup]](https://github.com/EddyIAM/Lab/blob/main/Docker%20setup.md)
[[Raspberry PI Setup]](https://github.com/EddyIAM/Lab/blob/main/Pi%20setup.md)

The lab setup is complete.  For now anyway.  As always things change right?

Primary purposes;

1. Allow me to play with Docker Swarm.
2. Allow me to play with [saltstack](https://saltproject.io/).
3. Allow me to play with an [OPNSense](https://opnsense.org/) Firewall.
4. Allow me to safely detonate malware and analyze it using [Security Onion](https://securityonionsolutions.com/), [osquery](https://osquery.io/), [velociraptor](https://docs.velociraptor.app/), and any other tools I want.
5. Prove to my Wife that our [internet](https://oss.oetiker.ch/smokeping/) is fine when her work blames it for connectivity issues.

The lab currently consists of;

- (10) Intel NUCs.
  - 8 Linux Ubuntu systems in a docker cluster
  - 1 Windows server
  - 1 Windows desktop  
- (1) Old gamer desktop
  - Runs Security onion in a VM and monitors everything coming out of the lab.
- (1) Laptop
  - Systems management
  - Flare VM
  - Sift/RemNUX VM
  - Parrot VM
- (5) Raspberry PIs
  - 1 SDR
  - 1 FlightAware
  - 1 Retro Pie
  - 2 Not currently doing anything
- (1) NAS for file storage
