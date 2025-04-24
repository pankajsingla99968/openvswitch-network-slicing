# 🧪 Open vSwitch Network Slicing Demo

This repository demonstrates how to create a virtual network environment using **Open vSwitch (OVS)**. It walks through installing OVS, setting up bridges, connecting veth pairs, assigning IPs, and simulating **network slices** using **VLANs**.

---

## 📦 Requirements

- Linux environment (tested on Ubuntu)
- Open vSwitch installed
- Root (sudo) privileges

---

## 🔧 What This Project Does

- Installs Open vSwitch
- Creates a virtual switch (`br-free5gc`)
- Sets up virtual Ethernet (veth) pairs
- Attaches interfaces to the switch
- Assigns IP addresses
- Demonstrates slicing using VLANs
- Performs ping tests to verify connectivity

---

## 🚀 Usage

You can copy the full script below and run it in your terminal or save it as `ovs-setup.sh`.

---

## 📝 Full Setup Script

```bash
#!/bin/bash

echo "[1/9] Installing Open vSwitch..."
sudo apt install openvswitch-switch -y

echo "[2/9] Creating OVS Bridge..."
sudo ovs-vsctl add-br br-free5gc

echo "[3/9] Creating veth pairs..."
sudo ip link add vethA type veth peer name vethA-peer
sudo ip link add vethB type veth peer name vethB-peer

echo "[4/9] Adding ports to bridge..."
sudo ovs-vsctl add-port br-free5gc vethA
sudo ovs-vsctl add-port br-free5gc vethB

echo "[5/9] Bringing interfaces up..."
sudo ip link set vethA up
sudo ip link set vethA-peer up
sudo ip link set vethB up
sudo ip link set vethB-peer up

echo "[6/9] Assigning IP addresses..."
sudo ip addr add 10.0.0.1/24 dev vethA-peer
sudo ip addr add 10.0.0.2/24 dev vethB-peer

echo "[7/9] Pinging between interfaces..."
ping 10.0.0.2 -c 4

echo "[8/9] Creating VLAN-based slices..."
sudo ip link add slice1-veth type veth peer name slice1-peer
sudo ip link add slice2-veth type veth peer name slice2-peer
sudo ovs-vsctl add-port br-free5gc slice1-veth tag=10
sudo ovs-vsctl add-port br-free5gc slice2-veth tag=20

echo "[9/9] Configuring VLAN interfaces..."
sudo ip link set slice1-peer up
sudo ip link set slice1-veth up
sudo ip link set slice2-peer up
sudo ip link set slice2-veth up
sudo ip addr add 192.168.10.1/24 dev slice1-peer
sudo ip addr add 192.168.20.1/24 dev slice2-peer
ping 192.168.10.1 -c 4
ping 192.168.20.1 -c 4

echo "✅ OVS setup and slicing completed!"
