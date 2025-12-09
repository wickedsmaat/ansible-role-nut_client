# ansible-role-nut_client

## Overview

This role configures a system as a **NUT UPS network client**, allowing it to receive power-event notifications and shutdown instructions from a central NUT master server.
A NUT client does **not** communicate with UPS hardware directly. Instead, it listens to the master server for status updates and executes coordinated shutdown actions when power events occur.
This ensures clean, predictable shutdown behavior across all homelab or distributed systems relying on the same UPS.

---

## Required Variables

This role **cannot run** unless the following variables are defined.
They represent the client’s NUT identity, server connection details, authentication credentials, and operating role.

| Variable                    | Description                                                |
| --------------------------- | ---------------------------------------------------------- |
| `ansible_nut_server_ip`     | IP address of the NUT master server                        |
| `ansible_ups_name`          | Name of the UPS as defined on the server                   |
| `ansible_ups_driver`        | Driver type used by the UPS (matches server configuration) |
| `ansible_ups_description`   | Human-readable description of the UPS                      |
| `ansible_ups_user`          | Username for NUT client authentication                     |
| `ansible_ups_user_password` | Password for the NUT user                                  |
| `ansible_nut_power_value`   | Shutdown power threshold used by upsmon                    |
| `ansible_nut_role`          | Client role (typically `slave` or `netclient`)             |

These must be set either:

* in the calling playbook
* in `group_vars` / `host_vars`
* or passed inline when including the role

Example:

```yaml
- name: Configure NUT clients
  hosts: lab_nodes
  vars:
    ansible_nut_server_ip: "192.168.50.10"
    ansible_ups_name: "ups01"
    ansible_ups_driver: "usbhid-ups"
    ansible_ups_description: "Primary Homelab UPS"
    ansible_ups_user: "upsmon"
    ansible_ups_user_password: "changeme"
    ansible_nut_power_value: 5
    ansible_nut_role: "netclient"

  roles:
    - ansible-role-nut_client
```

---

## Task Breakdown

### Installing the NUT Client Package

Installs `nut-client`, which provides the `upsmon` daemon responsible for maintaining the network link to the NUT master server and responding to UPS state transitions.

### Setting Client Configuration Variables

Captures all required UPS and server-specific values and prepares them for use within templates.
This ensures templates stay portable and reusable.

### Creating the NUT Configuration Directory

Ensures `/etc/ups` exists with correct ownership and permissions, providing a predictable location for all NUT configuration files.

### Deploying the `upsmon.conf` Configuration

Generates `/etc/ups/upsmon.conf` using the defined variables.
This defines authentication details, monitoring behavior, and shutdown actions.

### Configuring NUT Mode as `netclient`

Updates `/etc/ups/nut.conf` to place the machine in **network client mode** so it does **not** attempt direct hardware communication—only remote monitoring.

### Enabling and Starting the NUT Monitor Service

Ensures `nut-monitor` is enabled and running so the system continuously listens for UPS state changes and reacts appropriately.

---

## Summary

This role prepares any machine to act as a fully managed NUT network client.
By installing the required software, generating all necessary configuration files, and enabling UPS monitoring services, the role ensures the system participates effectively in a coordinated multi-machine shutdown strategy.

If you want, I can also generate a **schema**, **defaults file**, or **validation task** to ensure the required variables are present.
