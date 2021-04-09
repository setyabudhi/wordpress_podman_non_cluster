# wordpress_podman_non_cluster
# pull image wordpress and mariadb
> Sudo podman pull docker.io/library/wordpress

> sudo podman pull docker.io/library/mariadb

> sudo podman image ls

# create persisten volume
> mkdir -p /root/var/lib/mysql

> mkdir -p /root/var/www/html
> ls -lR var

# Create Virtual Private Network for WordPress and MariaDB Container
> sudo podman network ls

NETWORK ID NAME DRIVER SCOPE
ceb9d2e9cbad bridge bridge local
9d3c1b497be8 host host local
9da389311d9e none null local

# sudo podman network create private network
> sudo podman network create  mariadb-wp-privnet

90f36038fcaf6c19c598d0a7a6ddcc902d0af6f59dd2cdfed1e1df2d35eff02e

> sudo podman network ls

NETWORK ID NAME DRIVER SCOPE
ceb9d2e9cbad bridge bridge local
9d3c1b497be8 host host local
90f36038fcaf mariadb-wp-privnet bridge local
9da389311d9e none null local

# Run MariaDB and WordPress Container
> vi run_mariadb_with_persistent_volume_and_private_network.sh

#!/bin/sh
sudo podman container run -d \
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=redhat \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wordpress \
-e MYSQL_PASSWORD=redhat \
-v /root/var/lib/mysql:/var/lib/mysql:Z \
--network mariadb-wp-privnet \
-p 3306:3306 |
Docker.io/library/mariadb

> vi run_wordpress_with_persistent_volume_and_published_port.sh
 
#!/bin/sh
sudo podman container run -d \
--name wordpress \
-e WORDPRESS_DB_HOST=wordpressdb\ <--- this should using ip address if cannot container name
-e WORDPRESS_DB_USER=wordpress \
-e WORDPRESS_DB_PASSWORD=redhat \
-e WORDPRESS_DB_NAME=wordpress \
-v /root/var/www/html:/var/www/html:Z \
--network mariadb-wp-privnet \
-p 80:80 \
Docker.io/library/wordpress


> chmod +x run_mariadb_with_persistent_volume_and_private_network.sh
> chmod +x run_wordpress_with_persistent_volume_and_published_port.sh

> sudo ./run_mariadb_with_persistent_volume_and_private_network.sh
B596071feb0035fa3590860bc979a0769141a2de8b080235722ce0b2f1b3345c

> Sudo podman inspect wordpressdb | grep IPAddress

$ sudo podman inspect wordpressdb | grep IPAddress
            "IPAddress": "10.89.0.72",

# Run again vi for change WORDPRESS_DB_HOST= 10.89.0.72

> sudo ./run_wordpress_with_persistent_volume_and_published_port.sh

10f0489842c8887aba174399af3cb308885ae32cdc3e0603638192f250eee48a

> sudo podman container ls

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

10f0489842c8 wordpress "docker-entrypoint.s…" 2 seconds ago Up 2 seconds 0.0.0.0:80->80/tcp wordpress

b596071feb00 mariadb "docker-entrypoint.s…" 28 second ago Up 28 seconds 3306/tcp wordpressdb

# Open HTTP Port in Dynamic Firewall and Access WordPress
> sudo firewall-cmd --get-active-zones
public
interfaces: ens33

> sudo firewall-cmd --list-services

dhcpv6-client ssh

> sudo firewall-cmd --permanent --add-service=http
> sudo firewall-cmd –reload
> sudo firewall-cmd --list-services
dhcpv6-client http ssh

# To perform simple accessibility testing of WordPress
> curl http://localhost -sL |grep -i "wordpress\|installation\|werlcome"

