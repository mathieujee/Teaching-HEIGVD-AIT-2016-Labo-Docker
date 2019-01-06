# AIT Lab 04 - Docker

Authors: Mathieu Jee, Loïc Schürch

Date: December 2018

## Table of Contents

[TOC]

## Introduction



## Task 0: Identify issues and install the tools

- **M1:** 
- **M2:** 
- **M3:** 
- **M4:** 
- **M5**: 
- **M6:** 

Stats: 

![](C:\Users\Mathieu\Documents\HEIG-VD\Heig_3e\Semestre1\AIT\Labos\labo4\Teaching-HEIGVD-AIT-2016-Labo-Docker\report\Task0\1. Stats_Backend_Nodes.PNG)

URL of our repository: https://github.com/mathieujee/Teaching-HEIGVD-AIT-2016-Labo-Docker



## Task 1: Add a process supervisor to run several processes

Stats:

![](C:\Users\Mathieu\Documents\HEIG-VD\Heig_3e\Semestre1\AIT\Labos\labo4\Teaching-HEIGVD-AIT-2016-Labo-Docker\report\Task1\1. Stats_Backend_Nodes.PNG)



No particular difficulties encountered for this task. The main goal of this task was to configure our docker images to setup a process supervisor. The process supervisor (also called "*init system*") gives us the possibility to have multiple process on each container and, therefore, breaks the "one process per container" Docker philosophy.

For this lab, we use `S6-overlay` as process supervisor. With S6, we can also choose which process can restart, and wich process should exit the container. 



## Task 2: Add a tool to manage membership in the web server cluster

**1. Provide the docker log output for each of the containers: `ha`, `s1` and `s2`. You need to create a folder `logs` in your repository to store the files separately from the lab report. For each lab task create a folder and name it using the task number. No need to create a folder when there are no logs. **

See:

```
logs/task2/HA_logs.txt
logs/task2/S1_logs.txt
logs/task2/S2_logs.txt
```

**2. Give the answer to the question about the existing problem with the current solution. **

In our current solution, there is kind of misconception around the way we create the `Serf` cluster. The probleme is that if we run the webapps container first, it will fail because the `Serf` agent of `ha` is not available (webapps nodes will try to connect to the `Serf` cluster via `ha` container). If we start `ha` container first, it will fail because it will try to link the webapps container (that are not started yet). On the other hand, with this second approach, we can run webapps container successfully. 

To solve this issue, we have to modify the `Serf` command by replacing `--join` by `--retry-join`. 

- `-retry-join`: Address of another agent to join after starting up. This can be specified multiple times to specify multiple agents to join. If Serf is unable to join with any of the specified addresses, the agent will retry the join every `-retry-interval` up to `-retry-max` attempts. This can be used instead of `-join` to continue attempting to join the cluster. 

- `-retry-interval`:  Provides a duration string to control how often the retry join is performed. By default, the join is attempted every 30 seconds until success. This should use the "s" suffix for second, "m" for minute, or "h" for hour.
- `-retry-max`: Provides a limit on how many attempts to join the cluster can be made by `-retry-join`. If 0, there is no limit, and the agent will retry forever. Defaults to 0.

*Source:* https://www.serf.io/docs/agent/options.html

**3. Give an explanation on how `Serf` is working. Read the official website to get more details about the `GOSSIP` protocol used in `Serf`. Try to find other solutions that can be used to solve similar situations where we need some auto-discovery mechanism.**

`Serf` is a tool for cluster membership, failure detection, and orchestration that is decentralized, fault-tolerant and highly available.`Serf` relies on a lightweight protocol to communicate with every nodes of the cluster. Each `Serf Agent` send periodically messages to others `Serf Agents` to notify them of its status. Therefore, `Serf` is very useful and efficient to detect any failure inside nodes. It can also run custom scripts when custom events appear. These scripts can, for example, deploy or restart processes. 

The `GOSSIP` protocol used in `Serf` broadcasts messages to the cluster. Communication are done over UDP.  `GOSSIP` brings solutions to three major problems:

- Membership: `Serf` maintains cluster membership lists and is able to run custom scripts when a new node appears or an old one leaves. In our lab, `Serf` can maintain a list with our webapps for our load-balancer and notify it when a node comes online or goes offline.

- Failure detection and recovery: `Serf` automatically detects failed nodes within seconds, notify the whole cluster and can run custom scripts to handle these events. `Serf` will also try to reconnect to these nodes periodically.

- Custom event propagation: `Serf` can broadcast custom events and queries to the cluster to trigger deploys, propagate configuration, ... 

  

Other solutions that can be used to auto-discover nodes in cluster:

- Apache Ignite: offers two discovery mecasisms:
  - TCP/IP Discovery (designed and optimized for 100s of nodes deployments)
  - ZooKeeper Discovery (allows scaling Ignite clusters to 100s and 1000s of nodes preserving linear scalability and performance)



## Task 3: React to membership changes

**1. Provide the docker log output for each of the containers: `ha`, `s1` and `s2`. Put your logs in the `logs` directory you created in the previous task. **

See:

```
logs/task3/HA_logs_1.txt
logs/task3/HA_logs_2 (after running s1 only).txt
logs/task3/HA_logs_3 (after runing s1 and s2).txt
logs/task3/S1_logs.txt
logs/task3/S2_logs.txt
```

**2. Provide the logs from the `ha` container gathered directly from the `/var/log/serf.log` file present in the container. Put the logs in the `logs` directory in your repo. **

See: 

 ```
logs/task3/HA_logs_4 (custom log file).txt
 ```



## Task 4: Use a template engine to easily generate configuration files

**1. You probably noticed when we added `xz-utils`, we have to rebuild the whole image which took some time. What can we do to mitigate that? Take a look at the Docker documentation on [image layers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#images-and-layers). Tell us about the pros and cons to merge as much as possible of the command. In other words, compare:**

```
RUN command 1
RUN command 2
RUN command 3
```

vs.

```
RUN command 1 && command 2 && command 3
```

**There are also some articles about techniques to reduce the image size. Try to find them. They are talking about `squashing` or`flattening` images. **

In older version of Docker, it was important to minimize the number of layers to ensure their performance. Each time we use `RUN` command, it creates a new layer. By merging commands, we reduce the number of layers and improve the building speed.

"Squash" or "flatten" Docker images (or container) to make them smaller: 

- https://github.com/jwilder/docker-squash
- https://tuhrig.de/flatten-a-docker-container-or-image/
- https://www.codacy.com/blog/five-ways-to-slim-your-docker-images/



**2. Propose a different approach to architecture our images to be able to reuse as much as possible what we have done. Your proposition should also try to avoid as much as possible repetitions between your images. **





**3. Provide the `/tmp/haproxy.cfg` file generated in the `ha` container after each step. Place the output into the `logs` folder like you already did for the Docker logs in the previous tasks. Three files are expected.**

**In addition, provide a log file containing the output of the `docker ps` console and another file (per container) with `docker inspect <container>`. Four files are expected.**

See: 

```
logs/task4/haproxy.cfg.txt
logs/task4/haproxy.cfg (after running S1).txt
logs/task4/haproxy.cfg (after running S2).txt
```

and:

```
logs/task4/docker_ps.txt
logs/task4/docker_inspect_ha.txt
logs/task4/docker_inspect_S1.txt
logs/task4/docker_inspect_S2.txt
```



**4. Based on the three output files you have collected, what can you say about the way we generate it? What is the problem if any? **

The content of the file `haproxy.cfg` is overwritten. It only contains the id and the ip of the last node who joined the cluster. We should append data in the file instead of overwrite it.



## Task 5: Generate a new load balancer configuration when membership changes

**1. Provide the file `/usr/local/etc/haproxy/haproxy.cfg` generated in the `ha` container after each step. Three files are expected.**

See:

```
logs/task5/haproxy.cfg (after running ha).txt
logs/task5/haproxy.cfg (after running s1).txt
logs/task5/haproxy.cfg (after running s2).txt
```

**In addition, provide a log file containing the output of the `docker ps` console and another file (per container) with `docker inspect <container>`. Four files are expected.**

See:

```
logs/task5/docker_ps.txt
logs/task5/docker_inspect_ha.txt
logs/task5/docker_inspect_s1.txt
logs/task5/docker_inspect_s2.txt
```

**2. Provide the list of files from the `/nodes` folder inside the `ha` container. One file expected with the command output. **

See:

```
logs/task5/ls_nodes
```

**3. Provide the configuration file after you stopped one container and the list of nodes present in the `/nodes` folder. One file expected with the command output. Two files are expected.**

See:

```
logs/task5/haproxy.cfg (after stopping s1).txt
logs/task5/ls_nodes (after stopping s1).txt
```

**In addition, provide a log file containing the output of the `docker ps` console. One file expected.**

See: 

```
logs/task5/docker_ps (after stopping s1).txt
```

**4. (Optional:) Propose a different approach to manage the list of backend nodes. You do not need to implement it. You can also propose your own tools or the ones you discovered online. In that case, do not forget to cite your references. **

-

## Task 6: Make the load balancer automatically relaod the new configuration





## Difficulties





## Conclusion



