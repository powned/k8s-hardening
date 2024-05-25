# References

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

The Docker daemon relies on a [OCI](https://github.com/opencontainers/runtime-spec) compliant runtime (invoked via the containerd daemon) as its interface to the Linux kernel namespaces, cgroups, and __SELinux__.

[Secure your containers with SELinux](https://opensource.com/article/20/11/selinux-containers)

3. **Seccomp**

Seccomp is a sandboxing facility in the Linux kernel that acts like a firewall for system calls (syscalls). It uses Berkeley Packet Filter (BPF) rules to filter syscalls and control how they are handled. These filters can significantly limit a containers access to the Docker Host's Linux kernel - especially for simple containers/applications.

The `docker/no-chmod.json` file is a profile with the chmod(), fchmod(), and chmodat() syscalls removed from its whitelist.
