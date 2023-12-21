# Deprecation Warning: this is a personal project moved from another github Organization. The first commit of that project was on "Tue Apr 6 14:50:56 2021" and the last "Wed Jul 14 21:43:24 2021". At this moment (Wed Dec 20 2023) the project must be updated to the last software versions (ansible, vagrant, docker, k8s, benchs repositories, etc)

# Hardening Docker & Kubernetes - Ansible CIS Benchmark

Secure your Kubernetes cluster with the most good practices from [CIS](https://www.cisecurity.org/cis-benchmarks) in a automated way using Ansible.

Based on:

- [docker-bench-security](https://github.com/docker/docker-bench-security)

- [kube-bench](https://github.com/aquasecurity/kube-bench)

You can deploy a Kubernetes cluster completely functional for testing apps/services (increase VM resources in the [Vagrantfile](Vagrantfile) for custom needs). There are some manifests on [config/k8s/services](config/k8s/services) that you can use to provision your cluster. Be sure to add your installation logic in [tasks/services-k8s.yaml](./tasks/services-k8s.yaml). You can control this installation with corresponding variables in [vars/k8s.yaml](vars/k8s.yaml).

> Possible services you can install

- **Cert-Manager** (to review/update)

- **Nginx Ingress Controller** (to review/update)

- **ArgoCD** (to review/update)

> **TODO**

- Add script for creating TLS Docker certs automatically for being able to connect to the servers Docker daemon.

- Set variables with `envsub`

- Add more services for testing and hack.


> **New features for docker-bench-security** that you can see on [benchmark_docker](./benchmark_docker/)

- Ansible configuration.

- New argument `-v` for [benchmark/docker-bench-security.sh](./benchmark_docker/docker-bench-security.sh). You can pass the Docker official version and check if corresponds with yours.

```bash
$ sudo bash benchmark/docker-bench-security.sh -v 20.10.5
```

- New colours styles:
    - [WARN] in red for critical issues that you have to fix.
    - [INFO] in yellow for external (manually) actions (checks) you have to do in order to follow CIS recommendation. In general this check does not have a critical impact in your Docker environment nor may depend on external factors.
    - [NOTE] in blue for just a recommendations/suggestion or information to fix the point.

- [config/docker/daemon.json](config/docker/daemon.json) file configuration provided.

```json
{
    "cgroup-parent": "",          # CIS 2.9 SET your /foobar/path
    "log-level": "info",          # CIS 2.2
    "icc": false,                 # CIS 2.1
    "live-restore": true,         # CIS 2.13
    "userland-proxy": false,      # CIS 2.14
    "no-new-privileges": true,    # CIS 2.17
    "selinux-enabled": true,      # CIS 5.2
    "userns-remap": "default",    # CIS 2.8 BUG see troubleshooting
    "storage-driver": ""          # CIS 2.5 NOT "aufs"
}
```

- [docker/docker-compose.yaml](docker/docker-compose.yaml) file provided with examples and security options.

## Configuration / Requirements

> Test

[Vagrantfile](Vagrantfile) provided for testing Ansible playbooks.

- Install vagrant.

- Set required variables on [vars/](vars/) files.

Kubernetes cluster based on vagrant uses [flannel](https://github.com/flannel-io/flannel#flannel) or [calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart) as network plugin. See Troubleshooting to handle with possible errors.

> Ansible

You can deploy the configuration for hardening your machines

- Install ansible on local machine (where you are going to deploy from).

- Generate ssh keys and copy them on remote machines. See [keys/README.md](keys/README.md) and [create_user_ansible.sh](create_user_ansible.sh) for further information.

- Configure your hosts in [inventory/hosts](inventory/hosts) which ansible will connect to.

- Set required variables on [vars/](vars/) files. If your machines already has installed Docker then set `docker_configuration` to false.

Run `$ ansible all -m ping` for testing your configuration and check if ansible can connect to your machines.

> Manual benchmark

If you want to test manually if your cluster is securized...

- Copy [benchmark_docker](./benchmark_docker) folder to your remote machines.

- Copy the [config/docker/daemon.json](config/docker/daemon.json) to the Docker Daemon config path (by default `/etc/docker/daemon.json`) on your remote machines. You can add more options from [config/docker/daemon-template.json](./config/docker/daemon-template.json). **NOTE**: care about `"userns-remap"` option (see Troubleshooting part for further information).

- If you want to test the [docker/docker-compose.yaml](./docker/docker-compose.yaml) you have to copy it to another path. See [docker/README.md](./docker/README.md)

## Usage

> Test with Vagrant

- Install vagrant and run `$ vagrant up`. Now you have a Kubernetes cluster running in virtualbox.

```bash
$ vagrant status
$ vagrant ssh k8s-master
$ kubectl get nodes
$ journalctl -u kubelet
$ kubectl get po -n kube-system
```

> Ansible checks

Just run the following script [run_playbook.sh](./run_playbook.sh) for configuring and hardening Docker in your hosts.

```bash
$ bash run_playbook.sh playbooks/docker-k8s.yaml
```

Check benchmark logs on [benchmark_docker/results](./benchmark_docker/results) and [benchmark_k8s/results](./benchmark_k8s/results)

> Manual benchmark

**Please go to [docker-bench-security](https://github.com/docker/docker-bench-security) and [kube-bench](https://github.com/aquasecurity/kube-bench) for further information**

**docker-bench-security**

The CIS based checks are named `check_<section>_<number>`, e.g. `check_2_6` and community contributed checks are named `check_c_<number>`.

A complete list of checks is present in [functions_lib.sh](functions_lib.sh).

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -v 20.10.5` will run all checks and compare the Docker official version with your Docker enviroment.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -c check_2_2` will only run check `2.2 Ensure the logging level is set to 'info'`.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e check_2_2` will run all available checks except `2.2 Ensure the logging level is set to 'info'`.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e docker_enterprise_configuration` will run all available checks except the docker_enterprise_configuration group

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e docker_enterprise_configuration,check_2_2` will run all available checks except the docker_enterprise_configuration group and `2.2 Ensure the logging level is set to 'info'`

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -c container_images -e check_4_5` will run just the container_images checks except `4.5 Ensure Content trust for Docker is Enabled`

**kube-bench**

This proejct automate k8s benchmark using a go binary (kube-bench from [benchmark-k8s](./benchmark_k8s)). You can run

```bash
$ ./kube-bench --help
$ ./kube-bench --benchmark cis-1.6
```

For instance, with [benchmark-k8s/job.yaml](./benchmark_k8s/job.yaml) you will deploy a pod that bench the cluster

```bash
$ kubectl apply -f job.yaml
$ kubectl logs $pod_name
```

## Troubleshooting

> **Docker**

- You should make persistant every new rule you add with `auditctl` (see CIS 1). Check it after restart your environment. Ansible makes these default rules persistent but there may be an error if the paths/files do not exist.

- NOT to restart the Docker Daemon!! First stop it, make your config changes and finally start it.

```bash
$ systemctl stop docker # Stopping docker.service, but it can still be activated by docker.socket
$ systemctl stop docker.socket
```

- Fail to start a container with following ERROR

~~~
ERROR: for python  Cannot start service python: OCI runtime create failed: container_linux.go:367: starting container process caused: process_linux.go:495: container init caused: write sysctl key kernel.domainname: open /proc/sys/kernel/domainname: permission denied: unknown
~~~

> *bug*: [domainname denied if userns enabled](https://github.com/docker/for-linux/issues/743)

> *explanation*: [you can not set the domainname](https://github.com/opencontainers/runtime-spec/issues/592) just the hostname.

> *remmediation*: delete `"userns-remap": "default"` from [config/docker/daemon](config/docker/daemon.json)`

> *related to*: CIS 2.8, even `/etc/subuid` `/etc/subgid` are created.

> **Kubernetes**


1. Basic troubleshooting


```bash
$ kubectl get nodes -o wide
$ kubectl get po -n kube-system
```


2. Network Plugin


- The default CIDR range for **flannel** is `10.244.0.0/16`. If you are using `kubeadm init`, make sure to use `-–pod-network-cidr=10.244.0.0/16`. ***NOTE***: this project implement vagrant tests with `--pod-network-cidr=10.244.0.0/16` and the option `--iface=eth1` in [config/k8s/plugins/flannel/kube-flannel.yaml](./config/k8s/plugins/flannel/kube-flannel.yaml). Be sure to change this options if you want to modify the pod network.

- Nodes are in status *Ready* but **flannel** pods are on *Error* or *CrashLoopBackOff*

~~~
Error registering network: failed to acquire lease: node "node1" pod cidr not assigned
~~~

> *remmediation*: unless the cluster is initiallized with the `--pod-network-cidr` argument, sometimes it fails ([issue](https://github.com/kubernetes/kubeadm/issues/1899#issuecomment-552134904)). So you have to run

```bash
$ sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep -i cluster-cidr
    - --cluster-cidr=10.244.0.0/16
```

Which results is the subnet taken from [vars/k8s.yaml](/ars/k8s.yaml) and [Vagrantfile](./Vagrantfile). Then copy that CIDR and paste in the following command

```bash
$ for i in $(kubectl get nodes | grep node | awk '{print $1}'); do kubectl patch node $i -p '{"spec":{"podCIDR":"10.244.0.0/16"}}'; done
```


3. Services


- Sonarqube Error:

~~~
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
~~~

> *remmediation*: run `sysctl -w vm.max_map_count=26214` on the nodes.

> **Ansible**

~~~
ERROR! [DEPRECATED]: ansible.builtin.include has been removed. Use include_tasks or import_tasks instead. This feature was removed from ansible-core in a release after 2023-05-16. Please update your playbooks.
Ansible failed to complete successfully. Any error output should be
visible above. Please fix these errors and try again.
~~~

> *remmediation*: check your ansible version.

## References

**Docker**

[Docker Security (official github)](https://github.com/docker/labs/tree/master/security)

[Docker Security Options](https://docs.docker.com/engine/security/)

[Docker Compose Configuration](https://docs.docker.com/compose/compose-file/compose-file-v3/)

[Containers Security toolkit](https://www.stackrox.com/post/2017/08/hardening-docker-containers-and-hosts-against-vulnerabilities-a-security-toolkit/)

**Kubernetes**

[Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

[Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

[Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)

[Calico network plugin](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)

[Flannel network plugin](https://github.com/flannel-io/flannel#flannel)

**Extra**

Security features from CIS 5.

1. **AppArmor**

AppArmor (Application Armor) is a Linux Security Module (LSM). It protects the operating system by applying profiles to individual applications or containers. In contrast to managing capabilities with CAP_DROP and syscalls with seccomp, AppArmor allows for much finer-grained control. For example, AppArmor can restrict file operations on specified paths.

[AppArmor security profiles for Docker](https://docs.docker.com/engine/security/apparmor/)

[Protege contenedores con AppArmor](https://cloud.google.com/container-optimized-os/docs/how-to/secure-apparmor?hl=es)

2. **SElinux**

The Docker daemon relies on a [OCI](https://github.com/opencontainers/runtime-spec) compliant runtime (invoked via the containerd daemon) as its interface to the Linux kernel namespaces, cgroups, and __SELinux__

[Secure your containers with SELinux](https://opensource.com/article/20/11/selinux-containers)

3. **Seccomp**

Seccomp is a sandboxing facility in the Linux kernel that acts like a firewall for system calls (syscalls). It uses Berkeley Packet Filter (BPF) rules to filter syscalls and control how they are handled. These filters can significantly limit a containers access to the Docker Host's Linux kernel - especially for simple containers/applications.

The `docker/no-chmod.json` file is a profile with the chmod(), fchmod(), and chmodat() syscalls removed from its whitelist.
