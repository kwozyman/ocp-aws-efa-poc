Elastic Fabric Adapter support in Openshift
====

This is a proof of concept showing how to get working EFA support in Openshift.

Prerequisites
---

* a working Openshift cluster on Amazon Web Services with available kubeconfig
* `aws-cli` configured with proper AWS credentials

Node configuration
---

1. [Hugepages configured and reserved](https://docs.openshift.com/container-platform/4.8/scalability_and_performance/what-huge-pages-do-and-how-they-are-consumed-by-apps.html): `manifests/hugepages.yaml`
2. Memlock ulimits set to unlimited (hard and soft): `manifests/unlimited-memlock.yaml`
3. [Daemonset running in order to expose EFA capabilities](https://github.com/aws-samples/aws-efa-eks): `manifests/efa-k8s-device-plugin.yml`

Openshift configuration
---

4. [Node Feature Discovery operator](https://docs.openshift.com/container-platform/4.8/scalability_and_performance/psap-node-feature-discovery-operator.html)
5. Patched MPI operator (patched to keep using kubectl exec instead of [rs]sh): `manifests/mpi-operator.yaml`
6. Provide AWS ECR registry credentials (example for us-east-1 and us-west-2):
```
us_east_1_token=$(echo AWS:$(/usr/local/bin/aws ecr get-login-password --region us-east-1) | base64 -w0)
us-west-2_token=$(echo AWS:$(/usr/local/bin/aws ecr get-login-password --region us-west-2) | base64 -w0)
```
The registries used are `994408522926.dkr.ecr.us-east-1.amazonaws.com` and `602401143452.dkr.ecr.us-west-2.amazonaws.com`

Jobs configuration
---

7. Container images used for actual MPI: `container/Dockerfile-[intel|openmpi]`. Prebuilt images available at `quay.io/cgament/mpi:efa` and `quay.io/cgament/mpi:intel`
8. We are using [benchmarks for MPI over InfiniBand, Omni-Path, Ethernet/iWARP, and RoCE benchmarks from Ohio State Univeristy](https://mvapich.cse.ohio-state.edu/benchmarks/), specifically the `osu_latency`.
Example MpiJobs:
    * `jobs/mpi-tcp.yaml` -- an MPI job running over TCP, used to determine network connectivity between nodes
    * `jobs/mpi-support.yaml` -- will just show detected EFA support in the pod logs
    * `jobs/mpi-latency.yaml` -- latency benchmark using OpenMPI
    * `jobs/mpi-intel.yaml` -- latency benchmark using IntelMPI