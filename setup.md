change line in user-data by going to https://discovery.etcd.io/new and adding that line to the discovery line
This needs to happen everytime you create the cluster from scratch, you need a new token

execute lines to add right ssh keys for vagrant
vagrant ssh-config --host coreos-vagrant >> ~/.ssh/config
vagrant ssh-config --host coreos-vagrant | sed -n "s/IdentityFile//gp" | xargs ssh-add

The first command adds an entry to your ~/.ssh/config so you can ssh in using ssh coreos-vagrant
The second command adds the default insecure_private_key to your ssh-agent on your Mac

https://coreos.com/docs/launching-containers/launching/fleet-using-the-client/
export FLEETCTL_TUNNEL="$(vagrant ssh-config core-01 | sed -n "s/[ ]*HostName[ ]*//gp"):$(vagrant ssh-config core-01 | sed -n "s/[ ]*Port[ ]*//gp")"
echo $FLEETCTL_TUNNEL
fleetctl list-machines


sudo netstat -tulpn

added line
config.vm.network "forwarded_port", guest: 4001, host: (4001 + i - 1), auto_correct: true
to vagrantfile to expose etcd port to host machine
