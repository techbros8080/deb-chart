# Debezium Helm Chart

This chart bootstraps a cluster of Zookeeper - Kafka - Kafka Connect - VisualConnect

## Prerequisites

* Kubernetes 1.9.2+
* Helm 2.8.2+

## Docker Image Source

* [DockerHub -> TechBros8080](https://hub.docker.com/u/techbros8080)
* [DockerHub -> ConfluentInc](https://hub.docker.com/u/confluentinc/)

## Installing the Chart

### Install along with helm-charts

```console
git clone https://github.com/techbros8080/deb-chart.git
helm install deb-chart
```

To install with a specific name, you can do:

```console
helm install mydebezium deb-chart
```

### Install with a existing zookeeper

```console
helm install --set zookeeper.enabled=false,zookeeper.url="zooKeeper:2181" deb-chart
```

### Install with a existing kafka

```console
helm install --set zookeeper.enabled=false,kafka.enabled=false,kafka.bootstrapServers="PLAINTEXT://mykafka0:9092,PLAINTEXT://mykafka1:9092,PLAINTEXT://mykafka2:9092" deb-chart
```

There are
1. A [Confluent Zookeeper Ensemble](https://github.com/confluentinc/cp-helm-charts/tree/master/charts/zookeeper) created by zookeeper chart.
1. A [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) `boiling-heron-kafka` which contains 3 Kafka [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/): `boiling-heron-kafka-<0|1|2>`. Each Pod has a container running a Kafka Broker and an optional sidecar JMX Exporter Container.
1. A [Service](https://kubernetes.io/docs/concepts/services-networking/service/) `boiling-heron-kafka` for clients to connect to Kafka.
1. A [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) `boiling-heron-kafka-headless` to control the network domain for the Kafka processes.
1. A group(N = number of brokers) of [NodePort Service](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) `boiling-heron-kafka-${i}-external` to allow access to Kafka Cluster from outside.
1. A [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) which contains configuration for Prometheus JMX Exporter.
1. A [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) `kissing-macaw-cp-kafka-connect` which contains 1 Kafka Connect [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/): `kissing-macaw-cp-kafka-connect-6c77b8f5fd-cqlzq`.
1. A [Service](https://kubernetes.io/docs/concepts/services-networking/service/) `kissing-macaw-cp-kafka-connect` for clients to connect to Kafka Connect REST endpoint.

## Configuration

You can specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
helm install --name my-kafka -f my-values.yaml ./kafka
```

> **Tip**: A default [values.yaml](values.yaml) is provided

## Visual Connect

The configuration parameters in this section control the resources requested and utilized by the `techbros8080/dbz-kafka-connect` chart.

| Parameter         | Description                           | Default   |
| ----------------- | ------------------------------------- | --------- |
| `replicaCount`    | The number of Kafka Connect Servers.  | `1`       |

### Image

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `image` | Docker Images of Visual Connect. | `techbros8080/dbz-nginx, techbros8080/dbz-visual-connector, techbros8080/dbz-visual-connector-be` |
| `imageTag` | Docker Image Tags of Visual Connect. | `1.7.0` |
| `imagePullPolicy` | Docker Image Tags of Visual Connect. | `IfNotPresent` |

### Port

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `servicePort` | The port on which the Visual Connect will be available and serving requests. | `8084` |

### Volumes

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `volumes` | Volumes for visual connect container | see [values.yaml](values.yaml) for details |
| `volumeMounts` | Volume mounts for visual connect container | see [values.yaml](values.yaml) for details |

Visual Connects persistent volume must contains three folders;

-connectors: It can be Empty, or It can contain already prepared Connector definition json files.

-kafka-connect-servers: Should contains registered kafka-connect server definitions.

```json
    {
    "url": "http://deb-ora-kafka-connect:8083"
    }
```

-users: Can be empty, In order to register users, login to a running pod with visual-connector-be container, and use the following commands:

Users to login to Visual Connector should be defined under the `store/users` directory as json files. Inorder to add a user, you can execute the `add_user.sh` script which is at `scripts` folder.

```shell
scripts/add_user.sh -u admin -p adminpassword
```

## Kafka Connect Deployment

The configuration parameters in this section control the resources requested and utilized by the `techbros8080/dbz-kafka-connect` chart.

| Parameter         | Description                           | Default   |
| ----------------- | ------------------------------------- | --------- |
| `replicaCount`    | The number of Kafka Connect Servers.  | `1`       |

### Image

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `image` | Docker Image of Kafka Connect. | `techbros8080/dbz-kafka-connect` |
| `imageTag` | Docker Image Tag of Kafka Connect. | `1.7.0` |
| `imagePullPolicy` | Docker Image Tag of Kafka Connect. | `IfNotPresent` |

### Port

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `servicePort` | The port on which the Kafka Connect will be available and serving requests. | `8083` |

### Kafka Connect Worker Configurations

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `configurationOverrides` | Kafka Connect [configuration](https://docs.confluent.io/current/connect/references/allconfigs.html) overrides in the dictionary format. | `{}` |
| `customEnv` | Custom environmental variables | `{}` |

### Volumes

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `volumes` | Volumes for connect-server container | see [values.yaml](values.yaml) for details |
| `volumeMounts` | Volume mounts for connect-server container | see [values.yaml](values.yaml) for details |

### Secrets

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `secrets` | Secret with one or more `key:value` pairs | see [values.yaml](values.yaml) for details |

### Kafka Connect JVM Heap Options

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `heapOptions` | The JVM Heap Options for Kafka Connect | `"-Xms512M -Xmx512M"` |

### Resources

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `resources.requests.cpu` | The amount of CPU to request. | see [values.yaml](values.yaml) for details |
| `resources.requests.memory` | The amount of memory to request. | see [values.yaml](values.yaml) for details |
| `resources.requests.limit` | The upper limit CPU usage for a Kafka Connect Pod. | see [values.yaml](values.yaml) for details |
| `resources.requests.limit` | The upper limit memory usage for a Kafka Connect Pod. | see [values.yaml](values.yaml) for details |

### Annotations

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `podAnnotations` | Map of custom annotations to attach to the pod spec. | `{}` |

### JMX Configuration

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `jmx.port` | The jmx port which JMX style metrics are exposed. | `5555` |

### Prometheus JMX Exporter Configuration

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `prometheus.jmx.enabled` | Whether or not to install Prometheus JMX Exporter as a sidecar container and expose JMX metrics to Prometheus. | `true` |
| `prometheus.jmx.image` | Docker Image for Prometheus JMX Exporter container. | `solsson/kafka-prometheus-jmx-exporter@sha256` |
| `prometheus.jmx.imageTag` | Docker Image Tag for Prometheus JMX Exporter container. | `6f82e2b0464f50da8104acd7363fb9b995001ddff77d248379f8788e78946143` |
| `prometheus.jmx.imagePullPolicy` | Docker Image Pull Policy for Prometheus JMX Exporter container. | `IfNotPresent` |
| `prometheus.jmx.port` | JMX Exporter Port which exposes metrics in Prometheus format for scraping. | `5556` |
| `prometheus.jmx.resources` | JMX Exporter resources configuration. | see [values.yaml](values.yaml) for details |

### Running Custom Scripts

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `customEnv.CUSTOM_SCRIPT_PATH` | Path to external bash script to run inside the container | see [values.yaml](values.yaml) for details |
| `livenessProbe` | Requirement of `livenessProbe` depends on the custom script to be run  | see [values.yaml](values.yaml) for details |

### Deployment Topology

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `nodeSelector` | Dictionary containing key-value-pairs to match labels on nodes. When defined pods will only be scheduled on nodes, that have each of the indicated key-value pairs as labels. Further information can be found in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) | `{}`
| `tolerations`| Array containing taint references. When defined, pods can run on nodes, which would otherwise deny scheduling. Further information can be found in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) | `{}`

### Kafka Cluster

The configuration parameters in this section control the resources requested and utilized by the kafka chart.

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `brokers` | The number of Broker servers. | `3` |

### Image

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `image` | Docker Image of Confluent Kafka. | `confluentinc/cp-enterprise-kafka` |
| `imageTag` | Docker Image Tag of Confluent Kafka. | `6.1.0` |
| `imagePullPolicy` | Docker Image Tag of Confluent Kafka. | `IfNotPresent` |
| `imagePullSecrets` | Secrets to be used for private registries. | see [values.yaml](values.yaml) for details |

### StatefulSet Configurations

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `podManagementPolicy` | The Kafka StatefulSet Pod Management Policy: `Parallel` or `OrderedReady`. | `OrderedReady` |
| `updateStrategy` | The Kafka StatefulSet update strategy: `RollingUpdate` or `OnDelete`. | `RollingUpdate` |

### Confluent Kafka Configuration

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `configurationOverrides` | Kafka [configuration](https://kafka.apache.org/documentation/#brokerconfigs) overrides in the dictionary format | `{}` |
| `customEnv` | Custom environmental variables | `{}` |

### Persistence

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `persistence.enabled` | Whether to create a PVC. If `false`, an `emptyDir` on the host will be used. | `true` |
| `persistence.size` | Size for log dir, where Kafka will store log data. | `5Gi` |
| `persistence.storageClass` | Valid options: `nil`, `"-"`, or storage class name. | `nil` |
| `persistence.disksPerBroker` | The amount of disks that will be attached per instance of Kafka broker. | 1 |

### Kafka JVM Heap Options

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `heapOptions` | The JVM Heap Options for Kafka | `"-Xms512M -Xmx512M"` |

### Resources

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `resources.requests.cpu` | The amount of CPU to request. | see [values.yaml](values.yaml) for details |
| `resources.requests.memory` | The amount of memory to request. | see [values.yaml](values.yaml) for details |
| `resources.limits.cpu` | The upper limit CPU usage for a Kafka Pod. | see [values.yaml](values.yaml) for details |
| `resources.limits.memory` | The upper limit memory usage for a Kafka Pod. | see [values.yaml](values.yaml) for details |

### Annotations

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `podAnnotations` | Map of custom annotations to attach to the pod spec. | `{}` |

### JMX Configuration

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `jmx.port` | The jmx port which JMX style metrics are exposed. | `5555` |

### Prometheus JMX Exporter Configuration

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `prometheus.jmx.enabled` | Whether or not to install Prometheus JMX Exporter as a sidecar container and expose JMX metrics to Prometheus. | `true` |
| `prometheus.jmx.image` | Docker Image for Prometheus JMX Exporter container. | `solsson/kafka-prometheus-jmx-exporter@sha256` |
| `prometheus.jmx.imageTag` | Docker Image Tag for Prometheus JMX Exporter container. | `6f82e2b0464f50da8104acd7363fb9b995001ddff77d248379f8788e78946143` |
| `prometheus.jmx.imagePullPolicy` | Docker Image Pull Policy for Prometheus JMX Exporter container. | `IfNotPresent` |
| `prometheus.jmx.port` | JMX Exporter Port which exposes metrics in Prometheus format for scraping. | `5556` |
| `prometheus.jmx.resources` | JMX Exporter resources configuration. | see [values.yaml](values.yaml) for details |

### External Access

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `nodeport.enabled` | Whether or not to allow access to kafka cluster from outside k8s through NodePort. | `false` |
| `nodeport.servicePort` | The Port broker will advertise to external producers and consumers.  | `19092` |
| `nodeport.firstListenerPort` | The first NodePort that Kafka Broker will use for advertising to external producers and consumers. For each broker, advertise.listeners port for external will be set to `31090 + {index of broker pod}`. | `31090` |

### Deployment Topology

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `nodeSelector` | Dictionary containing key-value-pairs to match labels on nodes. When defined pods will only be scheduled on nodes, that have each of the indicated key-value pairs as labels. Further information can be found in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) | `{}`
| `tolerations`| Array containing taint references. When defined, pods can run on nodes, which would otherwise deny scheduling. Further information can be found in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) | `{}`

## Dependencies

### Zookeeper

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `zookeeper.enabled` | Whether or not to install zookeeper chart alongside kafka chart | `true` |
| `zookeeper.persistence.enabled` | Whether to create a PVC. If `false`, an `emptyDir` on the host will be used. | `true` |
| `zookeeper.persistence.dataDirSize` | Size for Data dir, where ZooKeeper will store the in-memory database snapshots. This will overwrite corresponding value in zookeeper chart's value.yaml | `5Gi` |
| `zookeeper.persistence.dataLogDirSize` | Size for data log dir, which is a dedicated log device to be used, and helps avoid competition between logging and snapshots. This will overwrite corresponding value in zookeeper chart's value.yaml. | `5Gi` |
| `zookeeper.url` | Service name of Zookeeper cluster (Not needed if zookeeper.enabled is set to true). | `""` |
| `zookeeper.clientPort` | Port of Zookeeper Cluster | `2181` |
