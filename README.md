# Running Podman on Mac



[Podman](https://podman.io/) is a tool for running Linux containers. You can do this from a MacOS desktop as long as you have access to a linux box either running inside of a VM on the host, or available via the network. You need to install the remote client and then setup ssh connection information.

[Multipass](https://multipass.run/) provides a command line interface to launch, manage and generally fiddle about with instances of Linux. The downloading of a minty-fresh image takes a matter of seconds, and within minutes a VM can be up and running.

Launch instances of Ubuntu and initialise them with cloud-init metadata like AWS, Azure, Google, IBM and Oracle clouds. Simulate your own cloud deployment on your workstation.

[HyperKit](https://github.com/moby/hyperkit) is an open-source hypervisor for macOS hypervisor, optimized for lightweight virtual machines and container deployment.

- Install [Multipass](https://multipass.run/) and Podman to your MacOSX:

```bash
$ brew install multipass podman 
```

- List Multipass available images:

```yaml
$ multipass find                                                                                           ░▒▓ ✔
Image                       Aliases           Version          Description
...
20.10                       groovy            20210130         Ubuntu 20.10
```

- Update the *cloud-config.yaml* file with your SSH public key.

```yaml
users:
  - name: ubuntu
    ssh-authorized-keys:
      - ssh-rsa AAAAB3N... user@example.com
...
```

- Rollout a new Ubuntu 20.10 instance using Cloud-Init definitions for instance first boot setup:

```bash
$ multipass launch -c 2 -m 2G -d 10G -n my-podman 20.10 --cloud-init ubuntu-20.10/cloud-config.yaml
Launched: my-podman

$ multipass list                                                                                      
Name                    State             IPv4             Image
...
my-podman               Running           192.168.64.2     Ubuntu 20.10
```

The above command will create a Ubuntu instance named ***my-podman\*** with 2 vCPUs, 2GB RAM and 10GB Storage. **Take note of the instance IPv4** (e.g. 192.168.64.2).

- Once the instance is created, check the status and jump into it:

```yaml
$ multipass shell my-podman
Welcome to Ubuntu 20.10 (GNU/Linux 5.8.0-41-generic x86_64)
...
ubuntu@podman-1:~$ podman info 
host:
  arch: amd64
  buildahVersion: 1.18.0
  cgroupManager: cgroupfs
...
  cpus: 2
  distribution:
    distribution: ubuntu
    version: "20.10"
...
  kernel: 5.8.0-41-generic
...
runRoot: /run/user/**1000**/containers
...
version:
  APIVersion: 2.1.0
  Built: 0
  BuiltTime: Thu Jan  1 01:00:00 1970
  GitCommit: ""
  GoVersion: go1.14.7
  OsArch: linux/amd64
  Version: 2.2.1
```

- Return to the MacOS prompt and your SSH private key to the system keyring. Add a new Podman connection informing the Podman's instance IPv4:

```yaml
$ ssh-add $HOME/.ssh/id_rsa

$ podman system connection add multipass \\
ssh://ubuntu@192.168.64.2/run/user/1000/podman/podman.sock
```

- List the Podman connections list:

```yaml
$ podman system connection list                                                                       ░▒▓ ✔
Name        Identity  URI
multipass*            ssh://ubuntu@192.168.64.2:22/run/user/1000/podman/podman.sock
```

- Gather once again the Podman stats:

```yaml
$ multipass shell my-podman
Welcome to Ubuntu 20.10 (GNU/Linux 5.8.0-41-generic x86_64)
...
ubuntu@podman-1:~$ podman info 
host:
  arch: amd64
  buildahVersion: 1.18.0
  cgroupManager: cgroupfs
...
  cpus: 2
  distribution:
    distribution: ubuntu
    version: "20.10"
...
  kernel: 5.8.0-41-generic
...
runRoot: /run/user/**1000**/containers
...
version:
  APIVersion: 2.1.0
  Built: 0
  BuiltTime: Thu Jan  1 01:00:00 1970
  GitCommit: ""
  GoVersion: go1.14.7
  OsArch: linux/amd64
  Version: 2.2.1
```

- Now, start a new container and enjoy:

```yaml
$ podman run -it busybox sh
# 
```