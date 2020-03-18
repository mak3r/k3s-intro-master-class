# Agenda
* k3s fundamentals 
* Use Case overview
* Single node demo
* HA Demo
* Multi-cluster management


# Setup a k3s cluster in HA mode for production use
* master (2 nodes) fedora 30
* worker (1) pi4B+ 4GB raspbian 64
* ?worker (3 nodes) pi3B+ 512 MB raspbian 64
* ?worker (1) pi zerow raspbian64
* data store (1) on my Mac mysql in a docker container
* network is configured with a GL-inet slate router
    - TODO: static IPs for all nodes 
    - dns is local only and is simply handled by the WAP routing table

--- 

# Prepare the MYSQL backing datastore
## Create an external DB
```
docker pull mysql:8.0.18
docker run --name=k3s-mysql -e MYSQL_ROOT_PASSWORD=k3s-sql-secret -p 3306:3306 -d mysql:8.0.18
```
## Configure the k3s sql admin
```
docker exec -it k3s-mysql mysql -uroot -p
CREATE USER 'k3s-admin'@'%' IDENTIFIED BY 'k3s-admin-pw';
GRANT ALL PRIVILEGES ON *.* TO 'k3s-admin'@'%';
```

# Prepare the postgresql datastore
## Create an external DB
```
docker run --name k3s-postgres --restart unless-stopped -e POSTGRES_PASSWORD=k3s-sql-secret -p 5432:5432 -d mak3r/k3s-postgres:12.2
```
## Check that the k3s db was created
```
docker exec -it k3s-postgres psql -h build-box -U postgres
select datname from pg_database;
```

# Single node dowload and run
* `curl -L https://github.com/rancher/k3s/releases/download/v1.17.0+k3s.1/k3s-arm64 -o k3s`
* `chmod +x ./k3s`
* `sudo ./k3s server -l k3s.log &`
* `sudo ./k3s kubectl get node`


# Launch server nodes

## for mysql backing store
Run this command on both nodes
* `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --write-kubeconfig-mode 644 --datastore-endpoint mysql://k3s-admin:k3s-admin-pw@tcp(mak3r:3306)/k3sdb -t agent-secret --tls-san mak3r.lan" INSTALL_K3S_VERSION="v1.17.3+k3s1" sh -`
## Alternate for setting server nodes as master only
* `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --datastore-endpoint mysql://k3s-admin:k3s-admin-pw@tcp(mak3r:3306)/k3sdb -t agent-secret --tls-san mak3r.lan --node-taint k3s-controlplane=true:NoExecute" INSTALL_K3S_VERSION="v1.17.3+k3s1" sh -`

## for postgresql backing store
Run this command on both nodes
* `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --write-kubeconfig-mode 644 --datastore-endpoint postgres://postgres:k3s-sql-secret@build-box:5432/k3sdb?sslmode=disable -t agent-secret --tls-san mak3r.lan" INSTALL_K3S_VERSION="v1.17.3+k3s1" sh -`
## Alternate for setting server nodes as master only
* `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --datastore-endpoint postgres://postgres:k3s-sql-secret@build-box:5432/k3sdb?sslmode=disable -t agent-secret --tls-san mak3r.lan --node-taint k3s-controlplane=true:NoExecute" INSTALL_K3S_VERSION="v1.17.3+k3s1" sh -`

# Configure the fixed registration address
* `docker run --name nginx-k3s-reg-addr -v /Users/markabrams/dev/k3s-advanced-class/nginx:/etc/nginx:ro -p 6443:6443 -p 443:443 -p 80:80 --restart unless-stopped -d nginx:stable`

# Join workers
* `curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent -t agent-secret --server https://master02.lan:6443" INSTALL_K3S_VERSION="v1.17.3+k3s1" sh -`

---

# Debugging
`show databases`
## Backup/restore the docker mysql container volume
```
docker create --volumes-from k3s-mysql --name k3s-mysql-2019-12-17 mysql:8.0.18
docker run  --volumes-from k3s-mysql-2019-12-15 -v $PWD:/backup:z busybox tar zcvf /backup/k3s-mysql-data-backup-8.0.18-2019-12-17.tar.gz /Users/markabrams/dev/k3s-advanced-class
docker run --name=k3s-mysql --restart unless-stopped --volumes-from k3s-mysql-2019-12-17 -e MYSQL_ROOT_PASSWORD=k3s-sql-secret -p 3306:3306 -d mysql:8.0.18
```
## Make sure the firewall is not blocking k3s traffic
```
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```

# TIPs
If you have already installed k3s and you decide to make a change to the underlying system - for example using a different CNI.
* Stop k3s `systemctl stop k3s`
* Run the new command
Note: this will cause you to lose your config data (VERIFY) and the whole cluster will need to be rebuilt

# Cleanup

```
sudo systemctl stop k3s
```

```
docker exec -it k3s-mysql mysql -uk3s-admin -p
drop database k3sdb;
```

# Extras
## Prepare a private registry for use with k3s
```
helm repo add harbor https://helm.goharbor.io
helm search repo harbor
```

## Import into Rancher

1. Open terminal
1. `tmux new`
1. login to all 4 nodes
1. insure local mysql is running
1. insure local nginx is running
1. 

# Killall k3s
for i in $(ps -ef | grep k3s | tr -s " " | cut -f 2 -d " "); do sudo kill -9 $i; done
