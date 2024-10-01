# Hi there 👋

<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->


# Project Structure

Raft/ 2PC

- fully functional raft is under sadds-raft main branch https://github.com/CMU-SV-DS/sadds-raft/tree/main
  - instructions: https://github.com/CMU-SV-DS/sadds-raft/blob/main/README.md
- fully functional 2PC is under sadds-raft refactor-structure-kejie branch (not mege into main yet) https://github.com/CMU-SV-DS/sadds-raft/tree/refactor-structure-kejie
  - Instructions: https://github.com/CMU-SV-DS/sadds-raft/blob/refactor-structure-kejie/StartInstagram.md

Data shardning

- 

Instagram client (for user)

- Backend is under sadds-service main branch https://github.com/CMU-SV-DS/sadds-service
- frontend is under sadds-client main branch https://github.com/CMU-SV-DS/sadds-client

- instructions: should run with raft/2pc, follow instructions here: https://github.com/CMU-SV-DS/sadds-raft/blob/main/README.md







# sadds-raft-suite

![raftdb workflow](https://github.com/CMU-SV-DS/sadds-raftdb/actions/workflows/build.yml/badge.svg)

This is an experimental project to build a distributed database based on the Raft
Consensus Algorithm. There are two main components when a database is launched.
The first one is Raft nodes which are storage entities based on the Raft Algorithm
to store data. The second is a Proxy server to monitor Raft nodes' status and handle
HTTP requests from clients. The following snippet shows how to build this project
and run the database via docker-compose.

## Suite Content
- Raft Core (`./src`): The main Raft implementation, which will be the node's runtime of the cluster.
- Raft Proxy (`./bin/proxy.cc`): The proxy server to handle HTTP requests from clients and get some Raft nodes' status.
- Raft Monitor (`./submodules/raft-monitor`): The monitor server to monitor Raft nodes' status and handle HTTP requests from admin-panel.
- Raft Client (`./submodules/raft-client`): The admin panel to monitor Raft nodes' status and list events and database data.
## Installation

```
git clone https://github.com/CMU-SV-DS/sadds-raft.git
git submodule init
git submodule update
```

❗ Also, make sure all the submodules are up-to-date, and on the `main` branch.

👀 If something doesn't work, try `git submodule update --init` to update the submodules.

## Build
### Compile Raft Core (Only do this if you know what you are doing)
```bash
# Raft core only: build with default options
$ ./build.sh -c -l -j 8

# Raft core only: build with proxy and docs
$ ./build.sh -D "-DBUILD_DOCS=ON -DBUILD_PROXY=ON" -c -l -j 8

# Raft core only: build with debug mode
$ ./build.sh -D "-DBUILD_DOCS=ON -DBUILD_PROXY=ON -DCMAKE_BUILD_TYPE=Debug" -c -l -j 8
```

### Run the whole suite inside Docker containers (recommended)
```bash
# Init raft config files (This should be done before building Docker) 
$ ./run.sh -i

# build via Docker
$ ./build.sh -d

# run raft proxy and nodes
$ ./run.sh -s

# terminate raft proxy and nodes
$ ./run.sh -tc
```

## Admin Panel

Allow users to monitor the status of the raft cluster and scale the cluster on the web page.

```bash
open http://localhost:3000
```

## Proxy Server

When the database is launched, a client can submit the following HTTP requests
to the Proxy to interact with the database. Note that the database adopt eventually
consistency model, which is guarantee that all accesses to a node will return
the last updated value.

```bash
$ export HOST="localhost:8080"
$ export NODE_ID=0

# check a node status by a node id
$ curl http://${HOST}/ping/${NODE_ID}
{"data":"","index":0,"leader":true,"success":true,"term":1}

# set a key/value pair to the database
$ curl -XPOST http://${HOST}/db/test -d "test"
{"data":"","index":1,"leader":true,"success":true,"term":1}

# get whole [k, v] pairs from the database
$ curl http://${HOST}/db/
{"data"::"{\"test\":\"test\"}","index":0,"leader":false,"success":true,"term":1}

# get a value from a key
$ curl http://${HOST}/db/test
{"data":"test","index":0,"leader":false,"success":true,"term":1}

# scale the cluster by adding 1 nodes
$ curl -XPATCH http://${HOST}/nodes
{"success":true,"adjustment":"1","exit_status":0}

# scale the cluster by removing 1 node
$ curl -XPATCH http://${HOST}/nodes -d "-1"
{"success":true,"adjustment":"-1","exit_status":0}

# scale the cluster by delete node of id = 2
$ curl -XDELETE http://${HOST}/nodes/2
{"success":true,"node_id":"2","exit_status":0}

# get the current scaling status or log
$ curl http://${HOST}/scale/log
# when scaling is in progress
{"message":"Scaling is in progress. Please try again later.","success":false}
# when adjustment scaling is done, log will be return
{"adjustment":"1","job_type":"adjust_node_count","log":"[INFO] Nodes: [0, 1, 2, 3]\n","success":true}
# when delete node is done, log will be return
{"job_type":"delete_node_by_id","log":"[INFO] Nodes: [1, 2, 3, 4, 5]\n","node_id":"0","success":true}

# get the current raft cluster configurations
$ curl http://${HOST}/configurations
{"data":"{\"heartbeat_duration\":50,\"random_duration\":500,\"timeout\":500}","index":0,"leader":false,"success":true,"term":1}

# set the current raft cluster configurations
$ curl -XPATCH 'http://${HOST}/configurations' \
--data '{
    "timeout": 100,
    "random_duration": 100,
    "heartbeat_duration": 100
}'
{"data":"","index":0,"leader":true,"success":true,"term":1}



```

## Python scripts

### `config/config_generator.py`
```bash
# Generate raft node config and docker-compose file for n nodes
$ python3 ./config/config_generator.py -n 3
$ python3 ./config/config_generator.py --nodes 3

# Add 3 nodes
$ python3 ./config/config_generator.py -a 3
$ python3 ./config/config_generator.py --adjust 3

# Remove 3 nodes
$ python3 ./config/config_generator.py -a -3
$ python3 ./config/config_generator.py --adjust -3

# Remove node of id = 3
$ python3 ./config/config_generator.py -r 3
$ python3 ./config/config_generator.py --remove 3
```

## Shell scripts - ./config/run.sh


```bash
./run.sh [OPTIONS]

# show help
$ ./run.sh -h

# init: create 3 (default) raft config files and create docker volume and network
$ ./run.sh -i

# start: start the raft service
$ ./run.sh -s

# terminate: stop raft service & remove all nodes
$ ./run.sh -t

# clean: remove docker volume and network
$ ./run.sh -c

# add: add 3 nodes & restart the raft cluster
$ ./run.sh -a 3

# remove: remove 3 nodes & restart the raft cluster
$ ./run.sh -a -3

# remove: remove node of id = 2 & restart the raft cluster
$ ./run.sh -r 2

# terminate and clean: stop raft service, remove all nodes, remove docker volume and network
$ ./run.sh -tc
```

## Reference

1. [In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf)
2. [The Raft Consensus Algorithm](https://raft.github.io/)
3. [gRPC Document](https://grpc.io/docs/)
4. [Drogon Document](https://drogon.docsforge.com/)
5. [spdlog Document](https://spdlog.docsforge.com/)
