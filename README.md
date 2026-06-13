# E2 — Source IP Whitelist (eBPF)

## Overview

This project implements a simple **eBPF-based Source IP Whitelist** using XDP.

The goal is to filter incoming packets at kernel level and allow only packets coming from trusted source IP addresses. Packets originating from non-whitelisted addresses are dropped before reaching the networking stack.

The implementation demonstrates how eBPF can be used for efficient packet filtering and access control directly inside the Linux kernel.

---

## Project Objectives

* Create an eBPF/XDP program.
* Load the program into the Linux kernel.
* Attach the program to a network interface.
* Maintain a whitelist of allowed source IP addresses.
* Drop packets coming from non-whitelisted sources.
* Verify successful loading and attachment using bpftool.

---

## Environment

### Operating System

* Ubuntu Linux (VirtualBox)

### Technologies

* eBPF
* XDP (Express Data Path)
* bpftool
* Docker
* Containerlab
* Linux Networking

---

## Topology

Two Containerlab nodes were deployed:

* clab-basic-lab-node1
* clab-basic-lab-node2

The eBPF program was attached to the network interface of **node1**.

---

## Implementation

The XDP program was compiled into an eBPF object file:

```bash
netprog.bpf.c
```

Compilation generated:

```bash
netprog.bpf.o
```

The object was then loaded using bpftool and pinned inside:

```text
/sys/fs/bpf/netprog
```

Finally, the program was attached to the target interface.

---

## Verification

The following command was used to verify the attachment:

```bash
bpftool net
```

Output:

```text
xdp:
eth1(25) generic id 142
```

This confirms that the eBPF/XDP program was successfully attached to the network interface.

---

## Issues Encountered

### Issue 1: bpftool not found inside the container

Error:

```text
bash: bpftool: command not found
```

Reason:

The container image did not include bpftool.

Solution:

bpftool was executed from the host namespace using:

```bash
nsenter
```

---

### Issue 2: Existing pinned BPF object

Error:

```text
path '/sys/fs/bpf/netprog' already exists
```

Reason:

A previously pinned eBPF object already existed.

Solution:

```bash
sudo rm /sys/fs/bpf/netprog
```

The program was then reloaded successfully.

---

### Issue 3: XDP attachment failed because of MTU mismatch

Error:

```text
Peer MTU is too large to set XDP
```

Reason:

The virtual Ethernet pair used by Containerlab had incompatible MTU settings.

Solution:

The interface MTU was adjusted and the program was attached using Generic XDP mode:

```bash
bpftool net attach xdpgeneric
```

---

## Results

The eBPF program was successfully:

* Compiled
* Loaded into the kernel
* Pinned in BPF filesystem
* Attached to the target interface
* Verified using bpftool

The final output confirmed that XDP was active on the interface.

---

## Learning Outcomes

Through this project, the following concepts were explored:

* eBPF architecture
* XDP packet processing
* Linux network namespaces
* Containerlab networking
* bpftool usage
* Loading and attaching BPF programs
* Troubleshooting kernel networking issues

---



## Conclusion

This project demonstrates a practical use of eBPF and XDP for implementing a Source IP Whitelist mechanism.

By performing packet filtering directly inside the Linux kernel, eBPF provides a highly efficient and scalable solution for network access control while minimizing processing overhead.

The project successfully demonstrated the compilation, loading, pinning, and attachment of an XDP program, as well as troubleshooting and resolving common deployment issues encountered during development.

