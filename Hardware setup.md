# Hardware Setup

## The Hardware

Each NUC is an Intel NUC5PPYB with 8GB of memory and a 120 GB SSD.

The Raspberry PIs vary.  

- (1) Raspberry Pi 4 Model B
- (3) Raspberry Pi 3 Model B
- (1) Raspberry Pi 2 Model B

## Windows Hardware Setup

The Windows systems are currently hand installed via USB.  

## Linux Hardware Setup

The Linux NUCs are deployed via Ubuntu MAAS.

## Raspberry PI Setup

The Raspberry PIs are all hand installed.

## The Network as a Whole

Most are connected to a 16 port switch which in turn connects to a MikroTek router.  A couple connect to the router directly.  Either way they are all on the lab subnet and behind the router.  

This router separates the lab from the production network and allows access to certain applications web interfaces and Guacamole via Traefik.

All lab traffic is monitored with Security Onion.
