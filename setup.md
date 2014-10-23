change line in user-data by going to https://discovery.etcd.io/new and adding that line to the discovery line
This needs to happen everytime you create the cluster from scratch, you need a new token
```
curl -w "\n" https://discovery.etcd.io/new
```

execute lines to add right ssh keys for vagrant
```
vagrant ssh-config --host coreos-vagrant >> ~/.ssh/config
vagrant ssh-config --host coreos-vagrant | sed -n "s/IdentityFile//gp" | xargs ssh-add
```

The first command adds an entry to your ~/.ssh/config so you can ssh in using ssh coreos-vagrant
The second command adds the default insecure_private_key to your ssh-agent on your Mac

[Fleet Using Client](https://coreos.com/docs/launching-containers/launching/fleet-using-the-client/)
```
export FLEETCTL_TUNNEL="$(vagrant ssh-config core-01 | sed -n "s/[ ]*HostName[ ]*//gp"):$(vagrant ssh-config core-01 | sed -n "s/[ ]*Port[ ]*//gp")"
echo $FLEETCTL_TUNNEL
fleetctl list-machines
```

```
sudo netstat -tulpn
```

added line
```
config.vm.network "forwarded_port", guest: 4001, host: (4001 + i - 1), auto_correct: true
```
to vagrantfile to expose etcd port to host machine

On Ec2 in the User Data (Advanced Details) on Step 3
```
#cloud-config

coreos:
  etcd:
    # generate a new token from https://discovery.etcd.io/new
    discovery: https://discovery.etcd.io/<token>
    # multi-region and multi-cloud deployments need to use $public_ipv4
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
  write_files:
    - path: /home/core/.dockercfg
      owner: core:core
      permissions: 0644
      content: |
        {
          "https://index.docker.io/v1/": {
            "auth": "xXxXxXxXxXx=",
            "email": "username@example.com"
          },
          "https://index.example.com": {
            "auth": "XxXxXxXxXxX=",
            "email": "username@example.com"
          }
        }
    - path: /etc/fleet/fleet.conf
      content: |
        public_ip="$private_ipv4"
        metadata="elastic_ip=true,public_ip=$public_ipv4"
```

Use Fleet on EC2
```
ssh-add <path to key to use>
ssh-add -l
fleetctl --tunnel=<ip.address.of.server> list-machines
```
1. First line adds agent
1. Second lists it
1. Third calls fleet on server

```
ssh-add -D
```
Removes agent

Docker container can connect to etcd (if on the same machine) by hitting ${COREOS_PRIVATE_IPV4} - as an env var passed in in service file

[Core OS Cluster Architectures](https://coreos.com/docs/cluster-management/setup/cluster-architectures/) - shows different architectures to use with CoreOS and hod to set them up.
