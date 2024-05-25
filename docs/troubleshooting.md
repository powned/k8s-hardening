# Troubleshooting

## **Docker**

- You should make persistant every new rule you add with `auditctl` (see CIS 1). Check it after restart your environment. Ansible makes these default rules persistent but there may be an error if the paths/files do not exist.

- NOT to restart the Docker Daemon!! First stop it, make your config changes and finally start it.

```bash
$ systemctl stop docker # Stopping docker.service, but it can still be activated by docker.socket
$ systemctl stop docker.socket
```

- Fail to start a container with following error:

~~~
ERROR: for python  Cannot start service python: OCI runtime create failed: container_linux.go:367: starting container process caused: process_linux.go:495: container init caused: write sysctl key kernel.domainname: open /proc/sys/kernel/domainname: permission denied: unknown
~~~

> *bug*: [domainname denied if userns enabled](https://github.com/docker/for-linux/issues/743)

> *explanation*: [you can not set the domainname](https://github.com/opencontainers/runtime-spec/issues/592) just the hostname.

> *remmediation*: delete `"userns-remap": "default"` from [config/docker/daemon](config/docker/daemon.json)

> *related to*: CIS 2.8, even `/etc/subuid` `/etc/subgid` are created.

## **Containerd**

- Sometimes kubeadm can not start and report the following error:

~~~~
[ERROR CRI]: container runtime is not running. Validate service connection: validate CRI v1 runtime API for endpoint unix:///var/run/containerd/containerd.sock
~~~~

> *remmediation*: It is necessary to delete the installation config for containerd. the ansible task "Copy containerd configuration" is implementd on [tasks/configure-docker.yaml](./tasks/configure-docker.yaml) fix the bug. Also review [config/containerd/README.md](./config/containerd/README.md)

## **Kubernetes**

1. Basic troubleshooting

```bash
$ kubectl get nodes -o wide
$ kubectl get po -n kube-system
```

- [Kubernetes Legacy Package Repositories Will Be Frozen On September 13, 2023](https://kubernetes.io/blog/2023/08/31/legacy-package-repository-deprecation/)

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

## **Ansible**

~~~
ERROR! [DEPRECATED]: ansible.builtin.include has been removed. Use include_tasks or import_tasks instead. This feature was removed from ansible-core in a release after 2023-05-16. Please update your playbooks.
Ansible failed to complete successfully. Any error output should be
visible above. Please fix these errors and try again.
~~~

> *remmediation*: check your ansible version.
