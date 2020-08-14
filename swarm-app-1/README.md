# Assignment: Create A Multi-Service Multi-Node Web App

## Goal: create networks, volumes, and services for a web-based "cats vs. dogs" voting app.
Here is a basic diagram of how the 5 services will work:

![diagram](./architecture.png)
- All images are on Docker Hub, so you should use editor to craft your commands locally, then paste them into swarm shell (at least that's how I'd do it)
- a `backend` and `frontend` overlay network are needed. Nothing different about them other then that backend will help protect database from the voting web app. (similar to how a VLAN setup might be in traditional architecture)
- The database server should use a named volume for preserving data. Use the new `--mount` format to do this: `--mount type=volume,source=db-data,target=/var/lib/postgresql/data`

Preface:
You should have 3 manager nodes in your network. To do this, I spun up 3 nodes instances on play-with-docker.com and ran the following:
In Node 1: `docker swarm init --advertise-addr <IP Address>`
In Node 2: `docker swarm join` command from the output of initializing swarm on Node 1.
In Node 3: `docker swarm join` command from the output of initializing swarm on Node 1.

Since I have Node 2 and 3 as worker nodes, with node 1 being the leader, I want to promote my two worker nodes with `docker node promote <nodeid>`.


CLI Commands:
```
docker network create -d overlay backend
docker network create -d overlay frontend
```

### Services (names below should be service names)
- vote
    - bretfisher/examplevotingapp_vote
    - web front end for users to vote dog/cat
    - ideally published on TCP 80. Container listens on 80
    - on frontend network
    - 2+ replicas of this container

CLI Commands:
```
docker service create --name vote -p 80:80 --network frontend --replicas 2 bretfisher/examplevotingapp_vote
```

- redis
    - redis:3.2
    - key/value storage for incoming votes
    - no public ports
    - on frontend network
    - 1 replica NOTE VIDEO SAYS TWO BUT ONLY ONE NEEDED

CLI Commands:
```
docker service create --name redis --network frontend redis:3.2
```

- worker
    - bretfisher/examplevotingapp_worker:java
    - backend processor of redis and storing results in postgres
    - no public ports
    - on frontend and backend networks
    - 1 replica

CLI Commands:
```
docker service create --name worker --network backend --network frontend bretfisher/examplevotingapp_worker:java
```

- db
    - postgres:9.4
    - one named volume needed, pointing to /var/lib/postgresql/data
    - on backend network
    - 1 replica
    - remember set env for password-less connections -e POSTGRES_HOST_AUTH_METHOD=trust

CLI Commands:
```
docker service create --name db --network backend --mount type=volume,source=db-data,target=/var/lib/postgresql/data --env POSTGRES_HOST_AUTH_METHOD=trust postgres:9.4
```

- result
    - bretfisher/examplevotingapp_result
    - web app that shows results
    - runs on high port since just for admins (lets imagine)
    - so run on a high port of your choosing (I choose 5001), container listens on 80
    - on backend network
    - 1 replica

CLI Commands:
```
docker service create --name result --network backend -p 5001:80 bretfisher/examplevotingapp_result
```

Seeing the results:
After running all the CLI commands, you are able to go to your browser and see the voting app by going to the IP Address of any of the three nodes.
And since I chose port 5001, I am able to go to \<IP Address\>:5001 to see the result app.