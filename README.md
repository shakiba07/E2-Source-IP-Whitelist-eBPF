# E2 – Source IP Whitelist using eBPF/XDP

## Overview

This project implements an IPv6 Source IP Whitelist using eBPF and XDP. The objective is to allow packets only from approved IPv6 source addresses and drop packets coming from addresses that are not present in a whitelist.

The whitelist is stored inside a BPF map and is dynamically populated from user space using **bpftool**, allowing the list of authorized addresses to be modified without recompiling the eBPF program.

The project demonstrates practical usage of XDP packet processing, BPF maps, IPv6 header parsing, and user-space interaction with eBPF programs.

---

# Objectives

The main goals of this project are:

* Implement packet filtering using XDP.
* Parse Ethernet and IPv6 headers.
* Maintain a whitelist of allowed IPv6 source addresses.
* Load whitelist entries dynamically from user space.
* Drop packets originating from non-whitelisted addresses.
* Verify correct functionality through network tests.

---

# Architecture

The solution consists of two main components:

## Kernel Space

An XDP eBPF program is attached to the network interface.

The program:

1. Receives packets at the XDP hook.
2. Parses the Ethernet header.
3. Checks whether the packet contains IPv6 traffic.
4. Extracts the source IPv6 address.
5. Looks up the address inside a BPF whitelist map.
6. Returns:

   * `XDP_PASS` if the address exists in the whitelist.
   * `XDP_DROP` otherwise.

## User Space

The whitelist is managed through `bpftool`.

User-space operations include:

* Creating and pinning BPF maps.
* Inserting IPv6 addresses into the whitelist.
* Inspecting map contents.
* Verifying correct program attachment.

---

# BPF Map Design

The whitelist is implemented using a hash map.

## Map Type

```c
BPF_MAP_TYPE_HASH
```

## Key

IPv6 source address (`struct in6_addr`)

## Value

```c
u8 allowed = 1;
```

A value of `1` indicates that the source IPv6 address is authorized.

---

# Packet Processing Logic

The packet processing workflow is shown below:

```text
Incoming Packet
       |
       v
Parse Ethernet Header
       |
       v
Is IPv6?
       |
   Yes v
Parse IPv6 Header
       |
       v
Extract Source Address
       |
       v
Lookup in Whitelist Map
       |
   +---+---+
   |       |
Found    Not Found
   |       |
   v       v
XDP_PASS XDP_DROP
```

---

# Compilation

The program is compiled using Clang and the Linux eBPF toolchain.

```bash
make
```

Compilation generates the eBPF object file that can be loaded into the kernel.

---

# Loading the Program

The compiled XDP program is attached to the target network interface.

Verification:

```bash
sudo bpftool net
```

Expected output confirms that an XDP program is attached to the interface.

---

# Populating the Whitelist

Whitelist entries are inserted from user space using `bpftool`.

Example allowed addresses:

```text
3fff:172:20:20::3
3fff:172:20:20::6
```

Map contents can be verified using:

```bash
sudo bpftool map dump id <MAP_ID>
```

Successful output shows both IPv6 addresses stored inside the whitelist map.

---

# Experimental Validation

## Test 1 – Whitelisted Address

Command:

```bash
docker exec clab-basic-lab-node1 ping6 3fff:172:20:20::3 -c 4
```

Result:

```text
4 packets transmitted
4 packets received
0% packet loss
```

### Observation

The source address exists in the whitelist map.

The eBPF program returns:

```c
XDP_PASS
```

The packet successfully reaches its destination.

---

## Test 2 – Second Whitelisted Address

Command:

```bash
docker exec clab-basic-lab-node1 ping6 3fff:172:20:20::6 -c 4
```

Result:

```text
4 packets transmitted
4 packets received
0% packet loss
```

### Observation

The address is present in the whitelist.

Traffic is accepted and forwarded normally.

---

## Test 3 – Non-Whitelisted Address

Command:

```bash
docker exec clab-basic-lab-node1 ping6 3fff:172:20:20::99 -c 4
```

Result:

```text
Destination unreachable
100% packet loss
```

### Observation

The address is not present in the whitelist map.

The eBPF program returns:

```c
XDP_DROP
```

The packet is discarded at the XDP layer before entering the normal kernel networking stack.

---

# Verification of Intermediate Requirement

The project satisfies the Intermediate requirement because:

* The whitelist is not hardcoded.
* IPv6 addresses are loaded dynamically from user space.
* `bpftool` is used to insert entries into the BPF map.
* Map contents can be modified without recompiling the program.
* Packet filtering decisions are performed using map lookups.

---

# Learning Outcomes

Through this project, the following concepts were explored:

* eBPF architecture
* XDP packet processing
* Linux networking internals
* IPv6 packet parsing
* BPF hash maps
* bpftool usage
* User-space and kernel-space interaction
* Dynamic access control using eBPF
* Network troubleshooting and debugging

---

# Conclusion

This project successfully implements an IPv6 Source IP Whitelist using eBPF and XDP.

The solution performs packet filtering directly at the XDP layer, providing efficient and early packet processing inside the Linux kernel. Whitelist entries are managed dynamically from user space through bpftool, making the system flexible and easy to maintain.

Experimental results confirm that packets originating from whitelisted IPv6 addresses are accepted, while packets from non-whitelisted addresses are dropped successfully.

The implementation fully satisfies the Basic and Intermediate requirements of the assignment and demonstrates a practical application of eBPF-based packet filtering.

