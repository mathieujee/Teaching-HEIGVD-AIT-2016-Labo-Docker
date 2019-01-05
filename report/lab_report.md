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





## Task 5: Generate a new load balancer configuration when membership changes





## Task 6: Make the load balancer automatically relaod the new configuration





## Difficulties





## Conclusion



