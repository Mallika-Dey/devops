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