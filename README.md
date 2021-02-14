# Running Podman on Mac Terminal

### How to run Podman commands directly from macOS terminal

I've found some articles explaining how to run Podman on top macOS [[1\]](https://developers.redhat.com/blog/2020/02/12/podman-for-macos-sort-of/), but the adopted approach was using *podman-machine*, which is a wrapper for an instance VM running on top of Virtulbox.

Since my goal was being able to run Podman commands directly from Mac terminal, I started looking for approaches alike the one I currently have for Docker (remote socket).

[Podman](https://podman.io/) is a tool for running Linux containers, same as we have for Docker. However, Podman out-of-the-box can only manage containers running on Linux instances, since it relies on an OCI compliant [Container Runtime](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.6yt1ex5wfo55) (runc, crun, runv, etc) to interface with the OS and create the running containers.

In a nutshell, we can do this from a macOS as long as you have access to a Linux box either running inside of a VM on the host or **available via the network**.

For running Linux on top of Mac, I've decided to use Multipass, rather than Virtualbox. [Multipass](https://multipass.run/) provides a CLI to launch and manage instances of Linux. The downloading of a minty-fresh image takes a matter of seconds, and within minutes a VM can be up and running. Multipass relies on [HyperKit](https://github.com/moby/hyperkit) which is an open-source hypervisor for macOS hypervisor.

At glance, the steps were:

1. Install Multipass and Podman on Mac;
2. Launch an Ubuntu 20.10 instance with cloud-init metadata same we do for AWS, Azure, GCP, etc;
3. Cloud-init will perform the minor customizations we need on the Linux instance (required packages and SSH settings);
4. Setup a Podman remote connection on Mac;
5. Enjoy the view.

### Getting the Hands Dirty

- Install [Multipass](https://multipass.run/) and Podman to your MacOSX:

```
$ brew install multipass podman
```



- List Multipass available images:

```
$ multipass find                                                                                           
Image                       Aliases           Version          Description
...
20.10                       groovy            20210130         Ubuntu 20.10
```



- Update the [*cloud-config.yaml*](https://raw.githubusercontent.com/hutger/podman-on-mac/main/ubuntu-20.10/cloud-init.yaml) file with your SSH public key.

```
...
users:
  - name: ubuntu
    ssh-authorized-keys:
      - ssh-rsa AAAAB3Nz... user@example.com
...
```



- Rollout a new Ubuntu 20.10 instance using *Cloud-Init* definitions:

```
$ multipass launch -c 2 -m 2G -d 10G -n my-podman 20.10 --cloud-init ubuntu-20.10/cloud-config.yaml
Launched: my-podman

$ multipass list                                                                                      
Name                    State             IPv4             Image
...
my-podman               Running           192.168.64.2     Ubuntu 20.10
```

The above command will create a Ubuntu instance named **my-podman** with 2 vCPUs, 2GB RAM and 10GB Storage. **Take note of the instance IPv4** (e.g. 192.168.64.2).



- Once the instance is created, jump into it and run the *podman info* for confirming the daemon status:

```
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
runRoot: /run/user/1000/containers
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



- Return to Mac terminal and after adding your SSH private key to the system keyring, create a new Podman connection informing the Podman’s instance IPv4:

```
$ ssh-add $HOME/.ssh/id_rsa

$ podman system connection add multipass \
ssh://ubuntu@192.168.64.2/run/user/1000/podman/podman.sock
```



- List the Podman connections list:

```
$ podman system connection list                                                                       
Name        Identity  URI
multipass*            ssh://ubuntu@192.168.64.2:22/run/user/1000/podman/podman.sock
```



- Run ***podman info\*** once again, but now from Mac terminal:

```
$ podman info 
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
runRoot: /run/user/1000/containers
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

```
$ podman run -it busybox sh
#
```



----

