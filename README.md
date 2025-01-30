
# Docker Networking

![image](https://github.com/user-attachments/assets/a7c23585-f68c-457e-8b2a-44e080ff418c)


This README serves as a basic guide to Docker networking, introducing fundamental concepts and practical commands to help you understand and test different networking modes. It covers essential Docker network types such as bridge, host, overlay, and macvlan, along with simple demonstrations to set up and experiment with container connectivity. Whether you're new to Docker networking or need a quick reference for testing setups, this guide provides a hands-on approach to learning and troubleshooting container networks.
<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;">Default Bridge Network</h2></summary>

**Step 1: Check docker Network** 

```bash
  docker network ls 
  docker network inspect bridge 
```

**Step 2: Start PostgreSQL Container**
Run PostgreSQL in detached mode:


```bash
  docker run -d --name postgres_db -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword postgres:latest  
```

**Step 3: Get PostgreSQL Container IP**
```bash
POSTGRES_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' postgres_db) 

echo "PostgreSQL IP: $POSTGRES_IP"
```

**Step 4: Start NGINX Container**

Run NGINX in detached mode and expose port 8080 

 
```bash
docker run -d --name nginx_web -p 8080:80 nginx:alpine 
```
 

**Step5: Check IP address of the containers**
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginx_web 
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' postgres_db 
```
 
 
**Step 6: Test Communication from NGINX to PostgreSQL**

Access the NGINX container shell: 
```bash
docker exec -it nginx_web sh 
```
 

Install PostgreSQL client tools (inside the NGINX container): 
```bash
apk update && apk add postgresql-client 
```
 
Test connectivity to PostgreSQL using its IP: 
```bash
psql -h 172.17.0.2 -U myuser -d postgres 
```
 

**Step 7: Test Communication from  PostgreSQL to ngnix**

Access the NGINX container shell: 
```bash
docker exec -it postgres_db sh 
```
 
Test connectivity to ngnix using its IP: 
```bash
curl http://172.17.0.3 
```

</details>

<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;">User Defined Bridge Network</h2></summary>


**Step 1: Create a Custom Bridge Network **
```bash
docker network create my-custom-net 
```
 

**Step 2: Run PostgreSQL Container on the Custom Network**
```bash
docker run -d --name postgres_db \
--network my-custom-net \ 
-e POSTGRES_USER=myuser \ 
-e POSTGRES_PASSWORD=mypassword \ 
-v postgres-data:/var/lib/postgresql/data \ 
postgres:latest 
```
 

**Step 3: Run NGINX Container on the Custom Network**

```bash
docker run -d --name nginx_web \ 
--network my-custom-net \ 
-p 8080:80 \ 
nginx:alpine 
```
 

**Step 4: Test Communication Using Service Names (DNS)**

From PostgreSQL to NGINX: 

Access the PostgreSQL container: 
```bash
docker exec -it postgres_db bash 
```
Install curl (Debian-based image): 
```bash
apt-get update && apt-get install -y curl 
```
Test connectivity to NGINX using its service name: 
```bash
curl http://nginx_web 
```
 

From NGINX to PostgreSQL: 

Access the NGINX container: 
```bash
docker exec -it nginx_web sh 
```
Install curl (Alpine-based image): 
```bash
apk add curl 
```
Test PostgreSQL connectivity (port 5432): 
```bash
curl -v telnet://postgres_db:5432 
```
 </details>

<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;"> None Network</h2></summary>


**Step 1: Run PostgreSQL on the None Network** 
```bash
docker run -d --name postgres_db --network none -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword postgres:latest 
```
 

**Step 2: Run NGINX on the None Network** 
```bash
docker run -d --name nginx_web --network none nginx:alpine 
```
 

**Step 3: Check Ip address** 
```bash
docker exec nginx_web ip addr 
```
 </details>

<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;"> ipvlan Network</h2></summary>


**Step 1: Check ip address and create an IPvlan Network** 
```bash
Ip addr 
```
Specify the parent interface (e.g., eth0, enp0s3, or your host’s physical NIC) and subnet. 

For IPvlan L2 Mode (Layer 2):  
```bash
docker network create -d ipvlan --subnet=10.0.2.0/24 --gateway=10.0.2.1 -o ipvlan_mode=l2 -o parent=enp0s3 my-ipvlan-net 
```
 

For IPvlan L3 Mode (Layer 3): 
```bash
docker network create -d ipvlan --subnet=10.0.2.0/24 -o ipvlan_mode=l3 -o parent=enp0s3 my-ipvlan-net 
```
 

**Step 2. Check if network is configured** 
```bash
Docker network ls 
```
 

**Step 3. Run Containers on the IPvlan Network** 

Assign static IPs from the subnet (or let Docker assign them dynamically). 

Example: NGINX Container with Static IP 
```bash
docker run -d --name nginx_web --network my-ipvlan-net nginx:alpine 
```
 

Example: PostgreSQL Container with Static IP 
```bash
docker run -d --name postgres_db --network my-ipvlan-net -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword postgres:latest 
```
 

**Step 4. Get the Ip address assigned to the containers** 
```bash
docker inspect postgres_db 
docker inspect nginx_web 
```
 

**Step 5. Test Connectivity** 

From the Host Machine: 
```bash
ping 10.0.2.2  # NGINX 
ping 10.0.2.3  # PostgreSQL 
```
From the Containers: 
```bash
docker exec -it nginx_web sh 
```
```bash
apk add postgresql-client 
psql -h 192.168.1.101 -U myuser -d postgres 
```
  </details>

<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;">  macvlan Network</h2></summary>

**Step 1.Check Ip Address and Create a Macvlan Network** 
```bash
ip addr
```

```bash
docker network create -d macvlan --subnet=10.0.2.0/24  --gateway=10.0.2.1 --ip-range=10.0.2.200/29 -o parent=enp0s3 my-macvlan-network 
```
**Step 2. Run Containers on the Macvlan Network** 

Assign static IPs within your subnet (avoid DHCP conflicts): 

NGINX Container: 
```bash
docker run -d --name nginx_web --network my-macvlan-net nginx:alpine 
```
PostgreSQL Container: 
```bash
docker run -d --name postgres_db --network my-macvlan-net -e POSTGRES_eUSER=myuser -e POSTGRES_PASSWORD=mypassword postgres:latest 
```

**Step 3. Test Connectivity** 

Ping NGINX 
```bash
ping 10.0.2.200 
```
Access NGINX via IP 
```bash
curl http://10.0.2.200 
```
Test PostgreSQL connectivity 
```bash
psql -h 10.0.2.201 -U myuser -d postgres 
```
**Step 4. Verify MAC Addresses** 
```bash
docker inspect postgres_db | grep -A 5 '"my-macvlan-net"' | grep '"MacAddress"' | awk -F'"' '{print $4}' 
docker inspect nginx_web | grep -A 5 '"my-macvlan-net"' | grep '"MacAddress"' | awk -F'"' '{print $4}' 
```

 </details>
 
<details>
   <summary markdown="span" style="cursor:pointer;"><h2 style="margin:0;"> Overlay Network</h2></summary>


For the demonstration setup, this guide will establish setup with two VM instances named host1 and host2. 

 

Please follow the following instructions  

 

Disable the offloading of IP checksum calculations for the network interface on host machine 1 

```bash
ip addr 
sudo ethtool -K ens33 tx-checksum-ip-generic off 
```

Disable the offloading of IP checksum calculations for the network interface on host machine 2 
```bash
ip addr 
sudo ethtool -K ens33 tx-checksum-ip-generic off 
```
Initialize docker swarm on host 1  
```bash 
docker swarm init
 ```
with the key join on the host machine 2
```bash 
docker swarm join ... 
```
 

Create overlay network on host 1 

```bash
docker network create -d overlay --attachable my-overlay-net 
docker network ls
``` 

```bash
Run container on host machine 1 
ocker run -d --name myapp --network my-overlay-net aputra/myapp-188:v3 
```

docker network ls on host 2 

```bash
Run container on host machine 2 
docker run -dit --name myapp-v2 --network my-overlay-net aputra/myapp-188:v3 
docker network ls 
docker exec -it myapp-v2 sh
```

Test connectivity 

```bash
curl myapp:8080/api/info 
```
 </details>


## Acknowledgements

 - [Docker Official Documentation](https://docs.docker.com/engine/network/)
 - [Docker Official Documentation](https://github.com/antonputra/tutorials/tree/main/lessons/188)


 
## License

[MIT](https://choosealicense.com/licenses/mit/)


