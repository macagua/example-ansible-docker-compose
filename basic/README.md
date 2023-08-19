# Docker containers for Ansible testing

Basic example of use Docker containers for Ansible testing.

## Install it

For download it, executing the following command:

```
git clone https://github.com/macagua/example-ansible-docker-compose.git
cd example-ansible-docker-compose
```

Start the Docker containers, executing the following command:

```
sudo docker compose up -d
```

Checkout if all three containers are up and running, executing the following command:

```
docker ps
```

## Access the containers

For access to the "controller" container, executing the following command:

```
docker exec -it controller /bin/sh
id
ansible --version
su ansible
cd $HOME/projects
pwd
exit
exit
```

For access to the "server1" container, executing the following command:

```
docker exec -it server1 /bin/sh
id
ansible --version
su ansible
cd $HOME/projects
pwd
ssh 172.19.0.4
exit
exit
exit
```

For access to the "server2" container, executing the following command:

```
docker exec -it server2 /bin/sh
id
ansible --version
su ansible
cd $HOME/projects
pwd
ssh 172.19.0.3
exit
exit
exit
```

---

## Configure the inventory

For the docker network list, executing the following command:

```
$ docker network ls
NETWORK ID     NAME                                DRIVER    SCOPE
43efdd9d1c0f   basic_default                       bridge    local
7e7506d20cb4   bridge                              bridge    local
bcee0d5ee370   host                                host      local
2d6ed3d85db6   none                                null      local
```

For the inspect the docker network "basic_default", executing the following command:

```
docker network inspect basic_default
```

For check the ip assigned for every docker containers, executing the following command:

```
for i in $(docker ps|awk '{print $1}'|tail -n +2); do docker exec $i ip a|grep 172.1;done
    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth0
    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth0
    inet 172.19.0.4/16 brd 172.19.255.255 scope global eth0
```

For create the inventory file, executing the following command:

```
cat <<EOF > inventory.txt
server1 ansible_port=22 ansible_host=172.19.0.3 ansible_user=root
server2 ansible_port=22 ansible_host=172.19.0.4 ansible_user=root
EOF
```

For list the server inventory, executing the following command:

```
ansible-inventory -i inventory.txt --list
```
---

## Configure the connection

For generating public/private ssh key pair, executing the following command:

```
docker exec -it controller /bin/sh
su ansible
ssh-keygen -f ~/.ssh/id_rsa_ansible
chmod 400 ~/.ssh/id_rsa_ansible
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_rsa_ansible
```

For copy the ssh pub key into every server, executing the following command:

```
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 22 root@172.19.0.3
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub -p 22 root@172.19.0.4
```

---

## Test the connection

For test the connection to servers inventory, executing the following command:

```
ansible -i ./inventory.txt all -m ping
ansible -i ./inventory.txt server1 -m ping
ansible -i ./inventory.txt server2 -m ping
ansible -i ./inventory.txt server* -m ping
ansible -i ./inventory.txt testing -m ping
ansible -i ./inventory.txt production -m ping
ansible -i ./inventory.txt testing:production -m ping
```

## Hello World from containers

For make a "Hello World" from Docker containers, executing the following command:

```
ansible -i ./inventory.txt all -m shell -a 'printf "Hello World from %s\n" $HOSTNAME'
ansible -i ./inventory.txt server1 -m shell -a 'printf "Hello World from %s\n" $HOSTNAME'
ansible -i ./inventory.txt server* -m shell -a 'printf "Hello World from %s\n" $HOSTNAME'
```

---

## Creating a Playbook

For create the "Hello World" playbook file, executing the following command:

```
cat <<EOF > playbook.yml
---
- hosts: all
  remote_user: root

  tasks:
  - name: "Test ping"
    ping:

  - name: "Print Hello World"
    shell: printf 'Hello World from %s\n' $HOSTNAME
    register: hello

  - name: "Output Print"
    debug: msg="{{ hello.stdout}}"
EOF
```

For executing the "Hello World" playbook file, executing the following command:

```
ansible-playbook -i ./inventory.txt ./playbook.yml
```
