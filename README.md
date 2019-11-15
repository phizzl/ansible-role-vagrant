# Vacker
A role for booting a **Va**grant machine and running a Do**cker** setup in it 

## Requirements
- Hosts should be bootstrapped for ansible usage (have python,...)
- Vagrant and Virtualbox must be installed (e.g. by using the roles `oefenweb.virtualbox` and `diodonfrost.vagrant`)

## Role Variables

| Variable | Description | Default value |
|----------|-------------|---------------|
| `vacker_src`| List of docker-compose setups **(see details!)**  | `[]` |


#### `vacker_src` details

The docker-compose list is used to define the setups to run or terminate per host.   Each item in
the list can have following attributes:

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `dest` | Destination to store the setup | yes | / |
| `state` | The setup state. It can be `up` for running the setup, `halt` for stopping the setup, or `destroy` for purging the setup | no | `up` |
| `authorized_keys` | The SSH keys that should get access to this machine and the remote host | no | / |
| `vagrantfile` | Will be injected to Vagrantfile in template `Vagrantfile.j2` | no | / |
| `provision_script` | Inline bash code to be executed when provisioning the VM | no | / |
| `boot_script` | Inline bash code to be executed every time when booting the VM | no | / |
| `forwarded_ports` | Port to be forwarded from the VM to the host. See `forwarded_ports` details. | no | [] |
| `key_file` | The SSH private key to set for the `vagrant` user in the VM | no | / |
| `flags` | Flags to pass to the `vagrant up` command, like `--provision` | no | / |
| `docker_compose_src` | See `docker_compose_src` details | yes | / |

#### `forwarded_ports` details

A list of port that are being forwarded from the guest to the host.

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `guest` | The port inside the VM that should be forwarded to the host | yes | / |
| `host` | The port on the host machine that should be used. If the port is being used Vagrant will try to autocorrect this by using a different port | no | `guest` |

#### `docker_compose_src` details
See `docker_compose_src` from package [`phizzl.dockercompose`](https://github.com/phizzl/ansible-role-docker-compose/blob/master/README.md#docker_compose_src-details).  
When provisioning the VM Ansible is installed inside the VM to bring up the docker-compose setup. 
To do so this package generates an Ansible configuration using `phizzl.dockercompose` to run Ansible inside the VM.  
The var `docker_compose_src` is a single item of the `docker_compose_src` from the package `phizzl.dockercompose`.  
The only var that's being set automatically is `dest` of an `docker_compose_src` item.
  
See the example playbook below.

## Dependencies
-

## Example Playbook  

```yaml
---
- hosts: all
  roles:
    - name: phizzl.vacker
      become_user: "{{ vacker_user }}"
      vars:
        vacker_src:
          - dest: /home/vacker/jenkins
            state: up
            key_file: "~/.ssh/id_rsa_vacker"
            authorized_keys:
              - "ssh-rsa ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890="
            docker_compose_src:
              repo: https://github.com/bitnami/bitnami-docker-jenkins.git
              inline: |
                version: '2'
                services:
                  jenkins:
                    image: 'bitnami/jenkins:2'
                    ports:
                      - '127.0.0.1:10000:8080'
                    volumes:
                      - 'jenkins_data:/bitnami'
                    environment:
                      - JENKINS_USERNAME=user
                      - JENKINS_PASSWORD=S3cretPassw0rd
                volumes:
                  jenkins_data:
                    driver: local
            forwarded_ports:
              - guest: 80
                host: 8080
```
