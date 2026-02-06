## Default Bridge 

```bash
# get ip of default docker bridge network
ip address show

# Run a container on the default bridge network in interactive mode
sudo docker run -itd --rm --name con1 busybox

# bridge network list
bridge link

# inspect bridge network
docker inspect bridge

# Enter inside a container and ping another container ip
docker exec -it con1 sh
ping ${ip_addr}
ip route  # for default bridge network gateway is docker0

# export container port to host port
docker run -itd --rm -p 90:80 --name ngcon1 nginx
```

## User Defined Bridge

```bash
# create bridge network
# This network has its own gateway, different from the default bridge
docker network create mallika-net

# Run a container on the user-defined bridge 
docker run -itd --rm --network mallika-net --name con3 busybox

# Inspect the user-defined bridge network
docker inspect mallika-net

## also support ping another container using container name
ping con4
```

## Host
```bash
# run on host network. no port mapping
# hit localhost:80 to get reponse
sudo docker run -itd --rm --network host --name ngcon1 nginx
```

## MACVLAN

Connect container directly to the physical network. Each container gets its own **Mac & Ip addresses**

### Bridge Mode
```bash
# create a macvlan network
docker network create -d macvlan \
--subnet 192.168.0.0/24 \
--gateway 192.168.0.1 \
-o parent=enp11s0 \
mallika-macvlan

# Run a container with a static IP
sudo docker run -itd --rm --network mallika-macvlan \
--ip 192.168.0.32 \
--name con3 busybox
```
Now, try to ping default gateway inside container con3. 
If it's a vm the ping might fail because <i>all mac addresses connected to same port</i>. Enable promiscuous mode in vm to solve this. 
Host cannot talk to its own macvlan containers. Then try to access from another device. 

### 802.1q Mode

```bash
# remove previous network
docker network rm mallika-macvlan

# create macvlan network on a VLAN sub-interface (802.1Q)
docker network create -d macvlan \
--subnet 192.168.0.0/24 \
--gateway 192.168.0.1 \
-o parent=enp11s0.20 \
mallika-macvlan20
```

## IPVLAN
### L2 (Default mode)

```bash
# create ipvlan network in L2 mode (default)
docker network create -d ipvlan \
--subnet 192.168.0.0/24 \
--gateway 192.168.0.1 \
-o parent=enp11s0 \
mallika-ipvlan

# run a container
docker run -itd --rm --network mallika-ipvlan \
--ip 192.168.0.32 \
--name con3 busybox
```

**Why host ↔ container does NOT work on bare metal**
Linux kernel drops packets that loop back to the same NIC.

```bash
Container → enp11s0 → Linux kernel → enp11s0 (same NIC)
```
Environment Comparison

| Scenario             | Who owns NIC | Hairpin allowed? | Host ↔ IPvlan works? |
| -------------------- | ------------ | ---------------- | -------------------- |
| Bare metal Ubuntu    | Linux kernel | No             | Fails              |
| VM Ubuntu on Windows | Hypervisor   | Yes            | Works              |
| Cloud VM (AWS/GCP)   | Hypervisor   | Yes            | Works              |

#### Macvlan vs IPvlan MAC Behavior

| Driver      | MAC per container          | Notes                  |
| ----------- | -------------------------- | ---------------------- |
| **Bridge**  | Virtual                    | NAT                    |
| **Macvlan** | Unique MAC per container | Looks like real device |
| **IPvlan**  | Same MAC for all         | Lightweight, scalable  |

### L3 Mode

A container can ping another container of different subnet.
```bash
# create network with 2 subnet
docker network create -d ipvlan \
--subnet 192.168.0.0/24 \
-o parent=enp11s0 -o ipvlan_mode=l3 \
--subnet 192.168.1.0/24 \
ipvlan-l3

# container in subnet 192.168.0.0/24
docker run -itd --rm --network ipvlan-l3 \
--ip 192.168.0.32 \
--name con3 busybox

# container in subnet 192.168.1.0/24
docker run -itd --rm --network ipvlan-l3 \
--ip 192.168.1.32 \
--name n-con3 busybox

# check routing table inside container
# gateway changed. not 192.168.0.0
docker exec -it con3 sh
ip route  
```
Changes in L3 mode:
1. Containers can ping each other across subnets
2. Docker creates an internal IPvlan router
3. Default gateway is NOT the real LAN gateway

```bash
con3 (192.168.0.32)
   ↓
Linux IPvlan router
   [no route to real LAN]
```

## None
No network interface (except loopback)

```bash
sudo docker run -itd --rm --network none --name con1 busybox
```