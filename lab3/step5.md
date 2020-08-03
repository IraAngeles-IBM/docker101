# Lab 3 - Introduction to Orchestration

## Step 5: Reconciliation

In the previous step, we updated the state of our service using `docker service update`. We saw Docker Swarm in action as it recognized the mismatch between desired state and actual state, and attempted to solve this issue.

The "inspect->adapt" model of docker swarm enables it to perform reconciliation when something goes wrong. For example, when a node in the swarm goes down it might take down running containers with it. The swarm will recognize this loss of containers, and will attempt to reschedule containers on available nodes in order to achieve the desired state for that service.

We are going to remove a node, and see tasks of our nginx1 service be rescheduled on other nodes automatically.

1. For the sake of clean output, first create a brand new service by copying the line below. We will change the name, and the publish port to avoid conflicts with our existing service. We will also add the `--replicas` command to scale the service with 5 instances.

    ```sh
    $ docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80  --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
    aiqdh5n9fyacgvb2g82s412js
    ```

1. On Node1, use `watch` to watch the update from the output of `docker service ps`. Note "watch" is a linux utility and might not be available on other platforms.

    ```sh
    $ watch -n 1 docker service ps nginx2
    ok
    ```

    This should result in a window that looks like this:

    ```sh
    Every 1s: docker service ps nginx1 2                                                                                              2017-05-12 15:29:20

    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STAT
    E            ERROR               PORTS
    6koehbhsfbi7         nginx2.1        nginx:1.12          node3               Running            Running 21 s
    econds ago
    dou2brjfr6lt        nginx2.2            nginx:1.12          node1               Running             Running 26 s
    econds ago
    8jc41tgwowph        nginx2.3            nginx:1.12          node2               Running             Running 27 s
    econds ago
    n5n8zryzg6g6        nginx2.4            nginx:1.12          node1               Running             Running 26 s
    econds ago
    cnofhk1v5bd8        nginx2.5            nginx:1.12          node2               Running             Running 27 s
    econds ago
    [node1] (loc
    ```

1. Click on Node3, and type the command to leave the swarm cluster.

    ```sh
    $ docker swarm leave
    ok
    ```

    This is the "nice" way to leave the swarm, but you can also kill the node and the following behavior will be the same.

1. Click on Node1 to watch the reconciliation in action. You should see that the swarm will attempt to get back to the declared state by rescheduling the containers that were running on node3 to node1 and node2 automatically.

    ```sh
    $ docker service ps nginx2
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
    jeq4604k1v9k        nginx2.1            nginx:1.12          node1               Running             Running 5 seconds ago
    6koehbhsfbi7         \_ nginx2.1        nginx:1.12          node3               Shutdown            Running 21 seconds ago
    dou2brjfr6lt        nginx2.2            nginx:1.12          node1               Running             Running 26 seconds ago
    8jc41tgwowph        nginx2.3            nginx:1.12          node2               Running             Running 27 seconds ago
    n5n8zryzg6g6        nginx2.4            nginx:1.12          node1               Running             Running 26 seconds ago
    cnofhk1v5bd8        nginx2.5            nginx:1.12          node2               Running             Running 27 seconds ago
    [node1] (loc
    ```

## Number of nodes

In this lab, our Docker Swarm cluster consists of one master, and two worker nodes. This configuration is not highly available. The manager node contains the necessary information to manage the cluster, so if this node goes down, the cluster will cease to function. For a production application, you will want to provision a cluster with multiple manager nodes to allow for manager node failures.

For manager nodes you want at least 3, but typically no more than 7. Managers implement the raft consensus algorithm, which requires that more than 50% of the nodes agree on the state that is being stored for the cluster. If you don't achieve >50%, the swarm will cease to operate correctly. For this reason, the following can be assumed about node failure tolerance.

* 3 manager nodes tolerates 1 node failure
* 5 manager nodes tolerates 2 node failures
* 7 manager nodes tolerates 3 node failures

It is possible to have an even number of manager nodes, but it adds no value in terms of the number of node failures. For example, 4 manager nodes would only tolerate 1 node failure, which is the same tolerance as a 3 manager node cluster. The more manager nodes you have, the harder it is to achieve a consensus on the state of a cluster.

While you typically want to limit the number of manager nodes to no more than 7, you can scale the number of worker nodes much higher than that. Worker nodes can scale up into the 1000's of nodes. Worker nodes communicate using the gossip protocol, which is optimized to be highly performant under large traffic and a large number of nodes.

If you are using [Play with Docker](http://play-with-docker.com), you can easily deploy multiple manager node clusters using the built in templates. Click the templates icon in the upper left to see what templates are available.

## Summary

In this lab, you got an introduction to problems that come with running container with production such as scheduling services across distributed nodes, maintaining high availability, implementing reconciliation, scaling, and logging. We used the orchestration tool that comes built-in to the Docker Engine- Docker Swarm, to address some of these issues.

Key Takeaways:

* The Docker Swarm schedules services using a declarative language. You declare the state, and the swarm attempts to maintain and reconcile to make sure the actual state == desired state
* Docker Swarm is composed of manager and worker nodes. Only managers can maintain the state of the swarm and accept commands to modify it. Workers have high scability and are only  used to run containers. By default managers can run containers as well.
* The routing mesh built into swarm means that any port that is published at the service level will be exposed on every node in the swarm. Requests to a published service port will be routed automatically to a container of the service that is running in the swarm.
* Many tools out there exist to help solve problems with orchestration containerized applications in production, include Docker Swarm, and the [IBM Kubernetes Service](https://cloud.ibm.com/kubernetes/catalog/create).
