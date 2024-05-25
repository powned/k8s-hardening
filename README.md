# Hardening Docker & Kubernetes with Ansible

Secure your Kubernetes cluster with the most good practices from [CIS](https://www.cisecurity.org/cis-benchmarks) in a automated way using Ansible.

### **Why?**

Common kubernetes administrators do not care about security concerns of their On Premise clusters. They use to think that the cluster is securized itself when running on an internal network, or maybe they delegate security concerns to the company security team (CISO). Also, cloud providers offers a kubernetes solution as a service (EKS, AKS, GKE...) but you do not know how these providers implement security features on the cluster you are using. So this solution applies automated checks to measure the security level of your cluster. You will see that it is perfect for running on an On Premise infrastructure due the analogies between deploying a On Premise k8s cluster ant the Vagrant environment of this repo. In this way you can use the Ansible playbooks not only to benchamrk docker and k8s but to configure the kubernetes into a machines network (masters and nodes).

Based on:

- [docker-bench-security](https://github.com/docker/docker-bench-security)

- [kube-bench](https://github.com/aquasecurity/kube-bench)

You can deploy a Kubernetes cluster completely functional for testing apps/services using `Vagrant` (increase VM resources in the [Vagrantfile](Vagrantfile) for custom needs). There are some manifests on [config/k8s/services](./config/k8s/services) that you can use to provision apps and services in your cluster. Be sure to add your installation logic in [tasks/services-k8s.yaml](./tasks/services-k8s.yaml). You can control this installation with corresponding variables in [vars/k8s.yaml](vars/k8s.yaml).

**Possible services you can install**

- Cert-Manager (to review/update)

- Nginx Ingress Controller (to review/update)

- ArgoCD

**New features for docker-bench-security** that you can see on [benchmark-docker](./benchmark-docker/)

- Ansible configuration.

- New argument `-v` for [benchmark/docker-bench-security.sh](./benchmark-docker/docker-bench-security.sh). You can pass the Docker official version and check if corresponds with yours.

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

**Software versions tested**

- Vagrant: v2.4.0
- Vagrant images: bento/ubuntu-18.04
- Ansible: core v2.16.1
- Kubernetes: v1.29
- Docker: v24.0.2 (build cb74dfc)
- Docker Compose: v1.17.1 (build unknown)
- Docker bench: v1.3.5 (modified by me)
- Kube bench: v0.7.0

### TODO

- Harden k8s cluster. This repo only run tests but not harden the cluster. Securize the k8s masters may be complicated to automate cause parameters variables between environments. Also, these manifests are very important so you may break the cluster if you make a mistake. You can take manifests examples located on [config/k8s/etc/](./config/k8s/etc/).

- Add script for creating TLS Docker certs automatically for being able to connect to the servers Docker daemon.

- Set variables with `envsub`

- Add more services for testing and hack.

## Configuration & Requirements

Go to [here](./docs/requirements.md)

## Usage

Go to [here](./docs/usage.md)

## Troubleshooting

Go to [here](./docs/troubleshooting.md)

## References

Go to [here](./docs/references.md)
