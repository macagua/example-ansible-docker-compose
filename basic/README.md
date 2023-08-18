# Docker containers for Ansible testing

Basic example of use Docker containers for Ansible testing

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
cd $HOME
pwd
ssh 172.19.0.3
exit
exit
```

For access to the "target1" container, executing the following command:

```
docker exec -it target1 /bin/sh
id
ansible --version
su ansible
cd $HOME
pwd
ssh 172.19.0.4
exit
exit
```

## Configure the inventory

For check the ip assigned for every docker containers, executing the following command:

```
for i in $(docker ps|awk '{print $1}'|tail -n +2); do docker exec $i ip a|grep 172.1;done
```

Create the inventory file, executing the following command:

```
cat <<EOF > inventory.txt
target1 ansible_port=2221 ansible_host=172.19.0.3 ansible_user=root
target2 ansible_port=2222 ansible_host=172.19.0.4 ansible_user=root
EOF
```

## Test the connection

For test the connection to servers inventory, executing the following command:

```
ansible target1 -m ping -i inventory.txt
ansible target2 -m ping -i inventory.txt
ansible target* -m ping -i inventory.txt
```
