# Configuration & Requirements

### Vagrant dev environment

[Vagrantfile](Vagrantfile) provided for testing Ansible playbooks.

- Install vagrant.

- Set required variables on [vars/](vars/) files.

Kubernetes cluster based on vagrant uses [flannel](https://github.com/flannel-io/flannel#flannel) or [calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart) as network plugin. See Troubleshooting to handle with possible errors.

### Ansible

You can deploy the configuration for hardening your machines

1. Install ansible on local machine (where you are going to deploy from).

2. Generate ssh keys and copy them on remote machines. See [keys/README.md](keys/README.md) and [create_user_ansible.sh](create_user_ansible.sh) for further information.

3. Configure your hosts in [inventory/hosts](inventory/hosts) which ansible will connect to.

4. Set required variables on [vars/](vars/) files. If your machines already has installed Docker then set `docker_configuration` to false.

Run `$ ansible all -m ping` for testing your configuration and check if ansible can connect to your machines.

### Manual benchmark

If you want to test manually if your cluster is securized:

1. Copy [benchmark-docker](./benchmark-docker) folder to your remote machines.

2. Copy the [config/docker/daemon.json](config/docker/daemon.json) to the Docker Daemon config path (by default `/etc/docker/daemon.json`) on your remote machines. You can add more options from [config/docker/daemon-template.json](./config/docker/daemon-template.json). **NOTE**: care about `"userns-remap"` option (see Troubleshooting part for further information).

3. If you want to test the [docker/docker-compose.yaml](./docker/docker-compose.yaml) you have to copy it to another path. See [docker/README.md](./docker/README.md)
