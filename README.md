# ansible-role-nut_client
Overview

This role configures a system as a NUT UPS client, allowing it to receive power-event notifications from a central NUT server.
A NUT client does not talk to UPS hardware directly; instead, it follows shutdown instructions from the master server.
This ensures clean, coordinated shutdown of homelab machines during power failures.

## Task-by-Task Breakdown
Installing the NUT Client Package

This task installs the nut-client package, which provides the upsmon daemon.
upsmon is responsible for maintaining a network connection to the UPS server and responding to UPS state changes.
This is the minimal software required for a NUT slave node.

Setting Client Configuration Variables

This task defines runtime variables such as UPS name, server address, credentials, and role designation.
These values feed into configuration templates used later.
By isolating them in one place, the role keeps templates portable and easy to update.

Creating the NUT Configuration Directory

This ensures /etc/ups exists with proper permissions.
Most NUT configuration files reside here, and missing this directory would prevent templates from rendering correctly.
This step guarantees a predictable filesystem layout for the rest of the role.

Deploying the upsmon Client Configuration

This task writes /etc/ups/upsmon.conf based on the previously defined variables.
The configuration instructs the system how to authenticate to the UPS server and what shutdown behavior to follow.
This is the core of NUT client behavior, enabling remote monitoring.

Configuring NUT Mode as “netclient”

This updates /etc/ups/nut.conf so the local system runs in netclient mode.
In this mode, the system does not access UPS hardware — it only listens to a remote NUT master.
This prevents conflicts and ensures the machine behaves purely as a slave.

Enabling and Starting the NUT Monitor Service

This activates the nut-monitor systemd unit so that upsmon starts at boot.
When enabled, the machine will continuously monitor UPS conditions from the remote server.
This ensures timely and controlled shutdowns during power events.

## Summary

The NUT client role transforms any system into a safely managed UPS-aware node.
It installs the necessary software, configures the client-side monitoring services, and ensures the system participates correctly in a coordinated multi-machine UPS shutdown strategy.