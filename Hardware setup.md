# Hardware Setup

## The Hardware

Each NUC is an Intel NUC5PPYB with 8GB of memory and a 120 GB SSD.

The Raspberry PIs vary.  

- (1) Raspberry Pi 4 Model B
- (3) Raspberry Pi 3 Model B
- (1) Raspberry Pi 2 Model B

## Windows Hardware Setup

The Windows desktop was hand installed via USB and then cloned using CloneZilla for easy re-imaging.

## Linux Hardware Setup

The Linux NUCs were initially deployed via Ubuntu MAAS but it's now offline so any new boxes, or rebuilds, will have to be done by hand.

## Raspberry PI Setup

The Raspberry PIs are all hand installed.

## The Network as a Whole

Most are connected to a 16 port switch which in turn connects to a MikroTek router.  A couple connect to the router directly.  Either way they are all on the lab subnet and behind the router.  

One port on the 16 port switch connects to an OPNSense firewall that has the detonation network on the other side.  The detonation network is completely isolated from the lab network except for access to/from Security Onion and velociraptor.  Currently this is where the Windows desktop resides.  

The MikroTex router separates the lab from the production network and allows access to certain applications web interfaces and Guacamole via Traefik.

All lab traffic is monitored with Security Onion.
