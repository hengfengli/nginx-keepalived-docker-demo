# nginx-keepalived-docker-demo
A small demo to run two nginx containers in active-passive mode by using keepalived and VIP. 

This demo is made to simulate a scenario with two running nginx servers, one as a master and another as a backup, in order to achieve high availability. 

There are some limitations in this demo:
* I run it in a single host. Normally, it would be better to create a scenario with two hosts. 
* I didn't spend time to map the virtual IP from docker network to the host. Instead, I use a haproxy to do this task. 
* Running both nginx and keepalived in an Alpine container is not easy. rc-service could not manage keepalived properly. I thought to use supervisor or circus, but they are a bit heavy in space. So I directly run keepalived as a background daemon. However, the drawback is if it fails, it will not restart (not robust). 

## How to run

Run the following command: 

```bash
$ docker-compose up -d
```

Now, visit `localhost:8000` and you would see `Primary`. 

```bash
$ docker ps
CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS              PORTS                    NAMES
42cc6637255b        nginxkeepaliveddockerdemo_nginx_slave    "/entrypoint.sh"         28 seconds ago      Up 24 seconds       80/tcp                   nginxkeepaliveddockerdemo_nginx_slave_1
3f39a7e356ce        haproxy:1.7-alpine                       "/docker-entrypoin..."   28 seconds ago      Up 25 seconds       0.0.0.0:8000->6301/tcp   nginxkeepaliveddockerdemo_proxy_1
6151af0d50db        nginxkeepaliveddockerdemo_nginx_master   "/entrypoint.sh"         28 seconds ago      Up 24 seconds       80/tcp                   nginxkeepaliveddockerdemo_nginx_master_1
```

Try to pause the master server: 

```bash
$ docker pause nginxkeepaliveddockerdemo_nginx_master_1
```

Now, visit `localhost:8000` and you should see `Secondary`. 

Recover the master server and pause the slave server: 
```bash
$ docker unpause nginxkeepaliveddockerdemo_nginx_master_1
$ docker pause nginxkeepaliveddockerdemo_nginx_slave_1
```

Visit `localhost:8000` and you should see `Primary` again. 

As you can see, when a master server is down, a backup server does automatic failover. 
