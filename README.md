
# Docker Networking


## Default Bridge Network

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


## User Defined Bridge Network

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
 
