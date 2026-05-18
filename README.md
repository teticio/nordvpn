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

Multiple invocations can run in parallel: sessions for the same country share one tunnel; different countries each get their own namespace and connection.

## How it works

For each country, the script creates a dedicated network namespace (`vpnns<N>`) connected to the host via a veth pair, brings up `OpenVPN` inside it, and runs your command in that namespace. It queries a NordVPN endpoint to pick the lowest-load server in the chosen country and downloads its `.ovpn` config.

Parallel invocations are coordinated through a session ref-count under `/run/nordvpn/countries/<country>/sessions/`: the first call sets the tunnel up, subsequent calls for the same country reuse it, and the last call to exit tears everything down (openvpn process, iptables rules, veth, namespace, DNS overlay, state dir, and `~/.nordvpn-cache/<country>/`). Sessions whose owner process is gone (e.g. `kill -9`) are garbage-collected on the next run.

State lives in `/run/nordvpn/` (cleared on reboot) and `~/.nordvpn-cache/` (openvpn config/log/pid — kept under `$HOME` because openvpn's AppArmor profile only allows reads/writes there).
