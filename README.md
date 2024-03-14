# kps-docker-swarm-configs

Repository for docker swarm configs for the kapeesa backend micro-services

# to access private docker registry images here are the steps

- Login to the private registry and the credentials will be saved in the docker config file

e.g login to github docker registry

# To successfully deploy the stack

- use the following command

```
    docker stack deploy --with-registry-auth -c <compose_file> <stack_name>
    sudo docker stack deploy --with-registry-auth -c docker-compose.yml tabz-media-services
    docker stack deploy --with-registry-auth -c docker-compose.prod.yml tabz-media-services
    docker stack deploy --with-registry-auth -c docker-compose.staging.yml tabz-media-services
```

when using docker stack deploy in a swarm the `--with-registry-auth` option will forward the login information to other node

```



```
