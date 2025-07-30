Build a 3 node Galera cluster locally
---------------------------------------------------------------------------------------------------------

Description:
------------------------------------------------
This exercise will help the understanding of Docker containers, Docker Compose and the orchestration of multi-container applications. Through configuring a mariadb Galera cluster, you will also learn database clustering, replication, and high availability strategies, gaining insights into maintaining data consistency and fault tolerance in distributed systems. The exercise also introduces practical aspects of network configuration within Docker, including network creation and inter-container communication, essential for designing scalable and secure applications. Additionally, the hands-on experience with volume management and custom configuration files underscores the importance of data persistence and fine-tuning applications to specific requirements.

Requirements

Set up the configuration file of each node (to be placed in the nodeX/mysql directory) such that, when all three nodes start, they form part of a three node Galera Cluster

Configuration and Setup Guide

Volumes: The volumes section for each service maps local directories to container paths. The local directories node1/data, node2/data, and node3/data are for persisting MySQL data, while node1/mysql, node2/mysql, and node3/mysql can store custom configuration files. You'll need to create these directories in your project folder.

Environment Variables: Adjust MYSQL_ROOT_PASSWORD, MYSQL_USER, MYSQL_PASSWORD, and MYSQL_DATABASE according to your needs. The Galera-specific variables configure the cluster. The GALERA_CLUSTER_BOOTSTRAP variable on mariadb-node1 initiates the cluster, while the other nodes join it.

Networking: The services are connected through a dedicated network named galera. This ensures they can communicate with each other as required for cluster operations.

Port Mapping: The first node maps port 3306 to the host. If you wish to access the other nodes directly from your host for testing, you'll need to map their ports to different host ports.

Networks Section: The networks section at the bottom of the file now includes additional configurations.

Driver: Specifies the network driver to use. bridge is the default and is suitable for standalone containers needing to communicate. It's also a good choice for a local Galera cluster setup.

IPAM: Stands for IP Address Management. The driver: default setting is usually sufficient, but you can customize the subnet to avoid conflicts with your local network or other Docker networks.

version: '3'

services:
  mariadb-node1:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
      MYSQL_DATABASE: exampledb
      GALERA_CLUSTER: "true"
      GALERA_CLUSTER_BOOTSTRAP: "true"
      GALERA_CLUSTER_ADDRESS: "gcomm://mariadb-node1,mariadb-node2,mariadb-node3"
    volumes:
      - ./node1/data:/var/lib/mysql
      - ./node1/mysql:/etc/mysql/conf.d
    ports:
      - "3306:3306"
    networks:
      - galera

  mariadb-node2:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
      MYSQL_DATABASE: exampledb
      GALERA_CLUSTER: "true"
      GALERA_CLUSTER_BOOTSTRAP: "false"
      GALERA_CLUSTER_ADDRESS: "gcomm://mariadb-node1,mariadb-node2,mariadb-node3"
    volumes:
      - ./node2/data:/var/lib/mysql
      - ./node2/mysql:/etc/mysql/conf.d
    depends_on:
      - mariadb-node1
    networks:
      - galera

  mariadb-node3:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
      MYSQL_DATABASE: exampledb
      GALERA_CLUSTER: "true"
      GALERA_CLUSTER_BOOTSTRAP: "false"
      GALERA_CLUSTER_ADDRESS: "gcomm://mariadb-node1,mariadb-node2,mariadb-node3"
    volumes:
      - ./node3/data:/var/lib/mysql
      - ./node3/mysql:/etc/mysql/conf.d
    depends_on:
      - mariadb-node1
    networks:
      - galera

networks:
  galera:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.22.0.0/16"





Docker Compose file reference: https://docs.docker.com/compose/compose-file/ 

Environment Variables in Compose: https://docs.docker.com/compose/environment-variables/ 

Volumes in Docker Compose: https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes 

Docker Networking: https://docs.docker.com/network/ 

Bridge Networking: https://docs.docker.com/network/bridge/ 

IPAM Configuration: https://docs.docker.com/compose/compose-file/compose-file-v3/#ipam 

Docker and Docker Compose Installation Guide: https://docs.docker.com/get-docker/  (Covers both Docker and Docker Compose installations)

