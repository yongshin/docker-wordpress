Docker Wordpress Example
=====================

A Wordpress and MySQL example that can deploy on Docker EE through the UI or CLI for both Swarm and Kubernetes orchestration.

## Kubernetes - Docker EE 2.0 Standard or Advanced

```
# Source client bundle
cd ucp-bundle-admin
source env.sh

# Create MySQL root password as Kubernetes secret
kubectl create secret generic mysql-root-password -n default --from-literal="password=wordpress"

# OPTIONAL: Verify secret is encrypted on disk on UCP manager node
docker exec -it -e ETCDCTL_API=3 ucp-kv etcdctl --endpoints https://127.0.0.1:2379 \
  --cacert /etc/docker/ssl/ca.pem --cert /etc/docker/ssl/cert.pem \
  --key /etc/docker/ssl/key.pem get /registry/secrets/default/mysql-root-password

# Deploy using k8s yaml file via CLI
cd ~/docker-wordpress/k8s
kubectl apply -f mysql-demo.yaml
kubectl apply -f wordpress-demo.yaml

# Destroy deployments
kubectl delete -f mysql-demo.yaml
kubectl delete -f wordpress-demo.yaml
```

## Swarm - Docker EE 2.0 Standard or Advanced

### Deploy using Docker Compose File using Secrets (Recommended)
```
# Source client bundle
cd ucp-bundle-admin
source env.sh

# Create Secrets

echo "mysql-root-password-secret" | docker secret create MYSQL_ROOT_PASSWORD -
shx8bnn6qkoqkqqans4pu1e8w

echo "mysql-password-secret" | docker secret create MYSQL_PASSWORD -
isn62qaix8fxnq9dojqtf466s

echo "wordpress" | docker secret create MYSQL_USER -
v2mz2i9ei5g9i3zv7cpy1o9am

echo "wordpress" | docker secret create MYSQL_DATABASE -
khoy0gkn334m4hapbmef12svj

# OR use a script to generate Secrets
./scripts/create-secrets.sh

docker secret ls
ID                          NAME                  CREATED              UPDATED
isn62qaix8fxnq9dojqtf466s   MYSQL_PASSWORD        About a minute ago   About a minute ago
khoy0gkn334m4hapbmef12svj   MYSQL_DATABASE        50 seconds ago       50 seconds ago
shx8bnn6qkoqkqqans4pu1e8w   MYSQL_ROOT_PASSWORD   About a minute ago   About a minute ago
v2mz2i9ei5g9i3zv7cpy1o9am   MYSQL_USER            About a minute ago   About a minute ago

# Deploy using Compose File
cd ~/docker-wordpress/swarm
export WORDPRESS_DOMAIN=wordpress.local
docker stack deploy -c docker-compose-secrets.yml wordpress
```

### Deploy using Docker Compose File without Secrets
```
# Source client bundle
cd ucp-bundle-admin
source env.sh

# Deploy using Compose File
cd ~/docker-wordpress/swarm
export WORDPRESS_DOMAIN=wordpress.local
docker stack deploy -c docker-compose-no-secrets.yml wordpress
```

## Swarm - Docker CE

### Deploy Wordpress using `docker stack` command with a Compose file

Run Wordpress on your local engine before deploying on production (Docker EE)

```
# Deploy using Compose File
cd ~/docker-wordpress/swarm
docker stack deploy -c docker-compose-ce.yml wordpress
```

### Deploy Wordpress using `docker service` commands

#### Deploy Services via Script
```
./scripts/run-wordpress-service.sh
```


#### Deploy Services manually
```
docker network create --driver overlay wordpress-network

docker service create --replicas 1 --network wordpress-network \
  --mount type=volume,destination=/data/db -e MYSQL_ROOT_PASSWORD=wordpress \
  -e MYSQL_DATABASE=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress \
  --name mariadb mariadb

docker service create --replicas 3 -p 80 --network ucp-hrm --network wordpress-network \
  -e WORDPRESS_DB_HOST=mariadb:3306 -e WORDPRESS_DB_PASSWORD=wordpress \
  --label com.docker.ucp.mesh.http=80=http://wordpress.david.dtcntr.net \
  --name wordpress wordpress
```
