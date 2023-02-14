# Tor OBFS4 Bridge Ansible Playbook

This is a quick-n-dirty ansible playbook to setup OBFS4 Tor bridges within a docker container. This way it is easy to deploy and destroy bridges on the go to circumvent blocking mechanisms based on bridge IPs as seen in russia by rotating bridge IPs.

The playbook uses the official docker container available from the Tor gitlab and works on Debian and Ubuntu.

## Preparation

- clone repo
- optional: create ansible inventory if deploying to multiple hosts at once
- edit env file in _files/configs_. Add at least a correct contact e-mail, other changes are optional. See https://community.torproject.org/relay/setup/bridge/docker/ for details
- Add ssh-pubkey files to _files/admin_pubkeys_

## Run the bridge

In case your provider does not support pre-distribution of ssh keys and only offers you an initial ssh-root-password use something like this which asks for the root password to push the ssh-keys configured above and subsequently deactivate root password login:

```shell
ansible-playbook -k -i <YOUR IP OR INVENTORY>, --extra-vars "ansible_user=root" init.yml
```

Afterwards the setup is creating the docker container and boots up the bridge.

And true, root is used for the kick-off process. As the container uses secure privilege seperation this seems to be an acceptable risk, espacialy for an instant bridge which should disappear after some time to rotate the IP. Of course you should secure your private ssh key with a password to keep the access to the server as tight as possible.

## Update the bridge

For now you should manually monitor the [`docker-obfs4-bridge`](https://gitlab.torproject.org/tpo/anti-censorship/docker-obfs4-bridge) Gitlab for updates. If there are any, you can quickly update your instances by issuing the following command (or similar according to your inventory)

```shell
ansible-playbook init.yml \
  -k -i <HOSTS TO UPDATE>, \
  --extra-vars "ansible_user=root" \
  --tags "update,tor"
```

## Post-Snap

The bridge needs some time to ramp up, check its availability etc. Afterwards you should be able to create a valid bridgeline using

```shell
 docker exec \
  $(docker ps --quiet --filter "name=docker-obfs4-bridge-obfs4-bridge-1") \
  get-bridge-line
```

Also consult this for more information post installation infos: https://community.torproject.org/relay/setup/bridge/post-install/
