# DPDK-P4 Buffer Overflow Management
## Table of Contents
- [Introduction](#Introduction)
- [Methods](#Methods)
- [Installation](#Installation)
- [License](#license)

## Introduction
This project involved the design and implementation of a software switch using DPDK (Data Plane Development Kit) that is configurable with 
P4 (Programming Protocol-Independent Packet Processors). 

<p align="center">
  <img src="https://github.com/guy-davidi/p4_project/assets/64970907/6e39b160-7df7-4856-b7b0-a6a2c92a6da9" alt="Image Description" />
</p>

## Methods
Our Integrated solution is comprised of two main stages:

A packet classifier that can be configured in real-time. The classifier is being loaded to the system in real-time using P4. The classifier assigns a Quality of Service (QoS) value to each packet.

Buffer-overflow management algorithm, Implemented in the NIC’s DPDK driver  (depends on the free buffer space and the packet’s  QoS).
Default algorithm – Every packet passes.
Three Buffers Round Robin – The buffer is split into three sub-buffers for handling packets with different priorities, and each sub-buffer is served in a round-robin manner.
Inverse Linear algorithm – Packets pass \ dropped deterministically (QoS threshold).
Probabilistic Algorithm – Packets are passes \ dropped based on a probability model:

<p align="center">
  <img src="https://github.com/guy-davidi/p4_project/assets/64970907/5ecdbee1-341f-4210-8013-1b80c0862189" alt="Image Description" />
</p>

## Results
We collected data of the total QoS passed through the switch and performed a competitive analysis of the different algorithms.
From our results we obtained, we observe a clear improvement of the default packet forwarding procedure in a high-stress network state.
Also, The switch can support different communication protocols, which can be configured in real-time

<p align="center">
  <img src="https://github.com/guy-davidi/p4_project/assets/64970907/284ad618-7084-4251-a8d6-a7d49ec45a19" alt="Image Description" />
</p>

## Installation
```
chmod 755 ./scripts/p4cinstall.sh
./scripts/p4c_install.sh
chmod 755 ./scripts/dpdk_install.sh
./scripts/dpdk_install.sh
```
## Build DPDK's Pipeline APP
```cd /home/user/dpdk/examples/pipeline
mkdir build
make
```
## DPDK's uses NICs, Bind DPDK driver to device
```
cd /home/user/dpdk/usertools
sudo ./dpdk-devbind.py -b vfio-pci 0000:01:00.0
sudo ./dpdk-devbind.py -b vfio-pci 0000:05:00.0
```
## Compilation l2fwd.P4 -> l2fwd.spec (which DPDK's pipeline uses)
```
p4c-dpdk --arch psa l2fwd.p4 -o l2fwd.spec 
```
## In order run l2fwd (example)
```
sudo su
cd  ~/p4_project
sudo ./dpdk-pipeline -c 0x3 -- -s l2fwd.cli
```
## Connecting CLI DPDK-Pipeline APP
```
telnet 0.0.0.0 8086
pipeline PIPELINE0 stats
```
## In order to add table entries last must be call to 'commit'
## Note that Class Of Service is given to a packet in addition to egress port.
```
pipeline PIPELINE0 stats
pipeline PIPELINE0 table ipv4_host add ipv4_host_table.txt
pipeline PIPELINE0 commit
pipeline PIPELINE0 table ipv4_host show
Table ipv4_host: key size 4 bytes, key offset 16, key mask [ffffffff], action data size 4 bytes
match 0x0A000000 action send port 0x1 class 1
match 0x0B652B58 action send port 0x0 class 2
Table ipv4_host currently has 2 entries.
```
## Showing the stats of the pipeline - output
```
Input ports:
        Port 0: packets 877005976 bytes 52620358560 empty 14523130228
        Port 1: packets 625173214 bytes 37510392840 empty 14774963007

Output ports:
        Port 0: packets 0 bytes 0 clone 0 clonerr 0
        Port 1: packets 48986172 bytes 2939170320 clone 0 clonerr 0
        DROP: packets 1453193035 bytes 66846879610 clone 0 clonerr 0

Tables:
        Table ipv4_host:
                Hit (packets): 48986184
                Miss (packets): 1453193035
                Action NoAction (packets): 1453193035
                Action send (packets): 48986184
                Action drop_1 (packets): 0
```
## Read QoS from DPDK-App CLI
```
pipeline PIPELINE0 regrd reg_counter_0 0
```
It can be seen that each sample has exact jump of 5 packets with QoS -> "17" that gave in the match action table!
## Wrtting QoS to hdr.ipv4.src_addr = class
<p align="center">
  <img src="https://user-images.githubusercontent.com/64970907/223533841-04f6a645-6a93-4692-97ac-1da1a9760678.png" alt="Image Description" />
</p>

## License
MIT License

Copyright (c) 2023 Guy Davidi & Amit Davidi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


