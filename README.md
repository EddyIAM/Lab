# Lab Setup

![Picture of current lab setup](HomeLab%20Pic.png "Lab")

[[Hardware Setup]](https://github.com/EddyIAM/Lab/blob/main/Hardware%20setup.md)
[[Linux NUC Setup]](https://github.com/EddyIAM/Lab/blob/main/Linux%20NUC%20setup.md)
[[Windows Setup]](https://github.com/EddyIAM/Lab/blob/main/Windows%20setup.md)
[[Docker Setup]](https://github.com/EddyIAM/Lab/blob/main/Docker%20setup.md)
[[Raspberry PI Setup]](https://github.com/EddyIAM/Lab/blob/main/Pi%20setup.md)

I haven't been able to play in the lab for the last 6+ months or so due to other projects.  Now that I have some time I'm going to start by patching up/rebuilding and documenting.  This time around I'm going to focus on some automation so I can rebuild/replace faster next time.  First up is using [saltstack](https://saltproject.io/).

Once the lab is back up and running it will be time to play.  On the list is catching up on new [Security Onion](https://securityonionsolutions.com/) features, [osquery](https://osquery.io/), and [velociraptor](https://docs.velociraptor.app/). 

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
