# Usage

### Vagrant dev environment

1. Install vagrant and run `$ vagrant up`. Now you have a Kubernetes cluster running in virtualbox.

```bash
$ vagrant status
$ vagrant ssh k8s-master
$ kubectl get nodes
$ journalctl -u kubelet
$ kubectl get po -n kube-system
```

### Ansible

Just run the following script [run_playbook.sh](./run_playbook.sh) for configuring and hardening Docker in your hosts.

```bash
$ bash run_playbook.sh playbooks/docker-k8s.yaml
```

Check benchmark logs on [benchmark-docker/results](./benchmark-docker/results) and [benchmark-k8s/results](./benchmark-k8s/results)

### Manual benchmark

**Please go to [docker-bench-security](https://github.com/docker/docker-bench-security) and [kube-bench](https://github.com/aquasecurity/kube-bench) for further information**

The below instructions have to be executed from the machine you want to test.

#### docker-bench-security

The CIS based checks are named `check_<section>_<number>`, e.g. `check_2_6` and community contributed checks are named `check_c_<number>`.

A complete list of checks is present in [functions_lib.sh](functions_lib.sh). Below you have some examples:

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -v 20.10.5` will run all checks and compare the Docker official version with your Docker enviroment.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -c check_2_2` will only run check `2.2 Ensure the logging level is set to 'info'`.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e check_2_2` will run all available checks except `2.2 Ensure the logging level is set to 'info'`.

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e docker_enterprise_configuration` will run all available checks except the docker_enterprise_configuration group

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -e docker_enterprise_configuration,check_2_2` will run all available checks except the docker_enterprise_configuration group and `2.2 Ensure the logging level is set to 'info'`

`$ bash docker-bench-security.sh -l /tmp/docker-bench-security.sh.log -c container_images -e check_4_5` will run just the container_images checks except `4.5 Ensure Content trust for Docker is Enabled`

#### kube-bench

This project automate k8s benchmark using the kube-bench go binary. Ansible will download it on the remote machine and then it will execute tests remotely. Manually you can run

```bash
$ ./kube-bench --help
$ ./kube-bench --benchmark cis-1.6
```

For instance, with [benchmark-k8s/job.yaml](./benchmark-k8s/job.yaml) you will deploy a pod that bench the cluster

```bash
$ kubectl apply -f job.yaml
$ kubectl logs $pod_name
```
