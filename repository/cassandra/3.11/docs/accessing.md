# Accessing Cassandra

This guide explains how to access a running KUDO Cassandra instance from your
application deployed within the same Kubernetes cluster.

:warning: The KUDO Cassandra operator currently does not support accessing the
Cassandra instance from outside of the same Kubernetes cluster.

## Pre-conditions

- KUDO Cassandra instance running
- KUDO CLI installed

## Steps

### Preparation

#### 1. Set the shell variables

The examples below assume that the instance and namespace names are stored in
the following shell variables. With this assumptions met, you should be able to
copy-paste the commands easily.

```bash
instance_name=cassandra
namespace_name=default
```

#### 2. Verify that the variables are set correctly

```bash
kubectl get instance $instance_name -n $namespace_name
```

Example output:

```bash
NAME        AGE
cassandra   16h
```

### Access Cassandra

You will run simple `cqlsh` command within an ephemeral pod on the Kubernetes
cluster to show how to connect to Cassandra.

#### 3. Retrieve the docker image name

In order to run `cqlsh` you need a container image which has it. For simplicity,
you can use the same image which is used by the cassandra nodes. Run the
following command to retrieve its full name:

```bash
cassandra_image=$(kubectl get pod -n ${namespace_name} ${instance_name}-node-0 --template '{{ (index .spec.containers 0).image }}{{"\n"}}')
echo ${cassandra_image}
```

Example output:

```
mesosphere/cassandra:3.11.5-0.1.2
```

#### 4. Run a command which accesses cassandra in a pod

This command illustrates what DNS name to use to connect to Cassandra:

```bash
kubectl run --wait cassandra-access-demo --image=${cassandra_image} --restart=Never -- \
 cqlsh --execute "describe cluster" ${instance_name}-svc.${namespace_name}.svc.cluster.local
```

Expected output:

```
pod/cassandra-access-demo created
```

#### 5. Retrieve the output

Once the pod completes (which should take no more than a few seconds), you can
see its output using a command such as the following:

```bash
kubectl logs cassandra-access-demo
```

Example output:

```
Warning: Cannot create directory at `/home/cassandra/.cassandra`. Command history will not be saved.


Cluster: cassandra1
Partitioner: Murmur3Partitioner
```

### Cleanup

#### 6. Delete the ephemeral pod

```bash
kubectl delete pod cassandra-access-demo
```

Expected output:

```
pod "cassandra-access-demo" deleted
```
