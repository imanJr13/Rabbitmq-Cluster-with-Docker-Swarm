# Rabbitmq-Cluster-with-Docker-Swarm
A Simple Giuide to cluster Rabbitmq with Docker Swarm
Setting up a RabbitMQ cluster with Docker Swarm and running instances on separate nodes involves several steps. Here's a detailed guide to achieve this:

Prerequisites
1. Docker and Docker Swarm installed on both nodes.
2. Nodes should be able to communicate with each other over the network.
3. Basic knowledge of Docker and Docker Swarm.

Step-by-Step Guide
1. Initialize Docker Swarm
On the first node, initialize Docker Swarm:
```
docker swarm init --advertise-addr <IP_OF_FIRST_NODE>
```

This will return a command to join other nodes to the swarm. It looks something like:

```
docker swarm join --token <TOKEN> <IP_OF_FIRST_NODE>:2377

```
2. Join the Second Node to the Swarm
On the second node, run the join command obtained from the first node:
```
docker swarm join --token <TOKEN> <IP_OF_FIRST_NODE>:2377

```

3. Create an Overlay Network
Create an overlay network that spans across both nodes:

```
docker network create --driver overlay --subnet 10.0.9.0/24 my_overlay_network

```
4. Create Docker Compose File
Create a docker-compose.yml file with the following content:

```
version: '3.8'

services:
  rabbitmq1:
    image: rabbitmq:3-management
    hostname: rabbit1
    networks:
      - rabbitmq_network
    environment:
      RABBITMQ_ERLANG_COOKIE: 'SWQOKODSQALRPCLNMEQG'
      RABBITMQ_NODENAME: 'rabbit@rabbit1'
      RABBITMQ_CLUSTER_PARTITION_HANDLING: 'autoheal'
    volumes:
      - rabbitmq1_data:/var/lib/rabbitmq
    deploy:
      placement:
        constraints: [node.hostname == <NODE1_HOSTNAME>]

  rabbitmq2:
    image: rabbitmq:3-management
    hostname: rabbit2
    networks:
      - rabbitmq_network
    environment:
      RABBITMQ_ERLANG_COOKIE: 'SWQOKODSQALRPCLNMEQG'
      RABBITMQ_NODENAME: 'rabbit@rabbit2'
      RABBITMQ_CLUSTER_PARTITION_HANDLING: 'autoheal'
    volumes:
      - rabbitmq2_data:/var/lib/rabbitmq
    deploy:
      placement:
        constraints: [node.hostname == <NODE2_HOSTNAME>]

networks:
  rabbitmq_network:
    external: true
    name: my_overlay_network

volumes:
  rabbitmq1_data:
  rabbitmq2_data:

```
Replace <NODE1_HOSTNAME> and <NODE2_HOSTNAME> with the actual hostnames of your nodes.

5. Deploy the Stack
Deploy the stack using the following command:

```
docker stack deploy -c docker-compose.yml rabbitmq_cluster

```
6. Form the RabbitMQ Cluster
Once the services are up and running, you need to connect the RabbitMQ nodes to form a cluster. Follow these steps:

 1.Access the RabbitMQ management console on the first node:

```
http://<NODE1_IP>:15672

``` 
Log in with the default username and password (guest / guest).

 2. Navigate to the "Admin" tab, then to the "Nodes" section.
 3. Repeat the process on the second node:

 ```
 http://<NODE2_IP>:15672

 ```
 4.On one of the nodes, run the following commands in the RabbitMQ container to join the cluster:

 ```
 docker exec -it <container_id_of_rabbitmq1> bash
 rabbitmqctl stop_app
 rabbitmqctl reset
 rabbitmqctl join_cluster rabbit@rabbit2
 rabbitmqctl start_app
 exit

 ```
 5. On the other node, run:

 ```
 docker exec -it <container_id_of_rabbitmq2> bash
 rabbitmqctl stop_app
 rabbitmqctl reset
 rabbitmqctl join_cluster rabbit@rabbit1
 rabbitmqctl start_app
 exit

 ```
7. Verify the Cluster

Check the RabbitMQ management console on either node. You should see both nodes listed under the "Nodes" section.

