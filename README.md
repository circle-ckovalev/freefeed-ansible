# Deploying FreeFeed

Assuming we are on Debian 10 as root.

Prerequisites:

    apg-get update && apt-get install gnupg man
    echo 'deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main' > /etc/apt/sources.list.d/ansible.list
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
    apt-get update && apt-get install ansible python3-pip

    echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list
    curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
    apt-get update && apt-get install postgresql

    rm -rf /var/lib/postgresql
    mkdir /mnt/volume_ams3_01/postgresql
    chown -R postgres:postgres /mnt/volume_ams3_01/postgresql
    ln -s /mnt/volume_ams3_01/postgresql /var/lib/
    systemctl restart postgres

    su - postgres
    psql
    postgres=# create role freefeed with login encrypted password '***';
    postgres=# create database freefeed owner freefeed;
    postgres=# grant all on database freefeed to freefeed;
    postgres=# \c freefeed
    postgres=# create extension if not exists pgcrypto;
    postgres=# create extension if not exists intarray;
    postgres=# create extension if not exists tablefunc;

    echo "listen_addresses = 'localhost,172.18.0.1'" > /etc/postgresql/12/main/conf.d/local.conf
    systemctl restart postgres

    apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
    apt-get update && apt-get install docker-ce docker-ce-cli containerd.io
    pip3 install docker

    echo "deb http://nginx.org/packages/debian $(lsb_release -cs) nginx" > /etc/apt/sources.list.d/nginx.list
    curl -fsSL https://nginx.org/keys/nginx_signing.key | apt-key add -
    apt-get update && apt-get install nginx

    apt-get install certbot python-certbot-nginx


Regular deploy:

    ansible-playbook -vv -i circle playbooks/server.yml -e freefeed_server_version='freefeed_release_1.84.2'
    ansible-playbook -vv -i circle playbooks/frontend.yml -e react_client_version='latest'

