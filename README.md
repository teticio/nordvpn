# NordVPN

## Introduction

Bash script to run a command in a NordVPN without affecting other applications using the network.

## Pre-requisites

You will need to have installed `openvpn`, `jq` and `uuid-runtime`. Depending on your distro, you may also need to install `curl`, `iproute2` and `iptables`.

## Usage

Create a file `.nordvpn.auth` in your home directory with your NordVPN username and password obtained from https://my.nordaccount.com/dashboard/nordvpn/manual-configuration/service-credentials/:

```bash
<username>
<password>
```

Then run a command with:

```bash
./nordvpn <country> <command>
```

For example:

```bash
./nordvpn Spain curl -s https://ipinfo.io/ip
```

## How it works

The script creates a network namespace called `vpnspace` to run both `OpenVPN` and your command. It queries a NordVPN endpoint to retrieve available servers and selects the best one in the chosen country (i.e., with the lowest load). It then downloads the corresponding `ovpn` configuration and connects using your credentials.

The network namespace will persist until the next reboot. After a reboot, the following configurations will remain:

- IP forwarding will be enabled; you can disable it by running `sudo sysctl net.ipv4.ip_forward=0`.
- A directory `/etc/netns/vpnspace` will exist and contain the DNS configuration. This directory can be safely deleted if needed.
