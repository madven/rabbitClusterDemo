## Rabbitmq Cluster (simple test with Java (send/receive))

### Required setup process 
(https://medium.com/@omerkarabacak/how-to-setup-rabbitmq-cluster-on-ubuntu-16-04-fed4f9a2afb4):

RabbitMQ needs Erlang to run so first of all we will install Erlang;

#### First of all, fixing Erlang version for RabbitMQ needs;

```
# edit with sudo vi /etc/apt/preferences.d/erlang and add this
Package: erlang*
Pin: version 1:20.1-1
Pin-Priority: 1000

Package: esl-erlang
Pin: version 1:20.1.7
Pin-Priority: 1000
```

* Install Erlang

```
$ wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
$ sudo dpkg -i erlang-solutions_1.0_all.deb
$ wget https://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc
$ sudo apt-key add erlang_solutions.asc
$ sudo apt-get update
$ sudo apt-get install erlang=1:20.1-1
```

* Add apt repository to source list

`$ echo "deb https://dl.bintray.com/rabbitmq/debian xenial main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list`

* Add release signing key

`$ wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -`

* Update apt repositories

`$ sudo apt-get update`

* Add host names of machines to each other. Sample /etc/hosts file;

```
10.0.0.10 rabbitmqnode1.rabbitmqcluster rabbitmqnode1
10.0.0.11 rabbitmqnode2.rabbitmqcluster rabbitmqnode2
10.0.0.12 rabbitmqnode3.rabbitmqcluster rabbitmqnode3
```

* Install RabbitMQ

`$ sudo apt-get install rabbitmq-server`

* Start and enable RabbitMQ Service

```
$ sudo systemctl start rabbitmq-server
$ sudo systemctl enable rabbitmq-server
```

* Enable RabbitMQ Management Console

`$ sudo rabbitmq-plugins enable rabbitmq_management`

#### Setup RabbitMQ Cluster

* After that we will setup RabbitMQ cluster;

```
# Erlang cookie from master Rabbitmq node
$ sudo cat /var/lib/rabbitmq/.erlang.cookie
# Add Erlang cookie that you get step before from master node to slave nodes with this command
$ sudo sh -c "echo '<ERLANG_COOKIE>' > /var/lib/rabbitmq/.erlang.cookie"
```

* Before this step, you have to reboot your slave servers. Run this at all slave nodes;

```
$ sudo rabbitmqctl stop_app;
$ sudo rabbitmqctl reset;
$ sudo rabbitmqctl join_cluster --ram rabbit@<MASTER_HOSTNAME>;
$ sudo rabbitmqctl start_app;
```

* Verify the cluster with running this command at master node;

`$ sudo rabbitmqctl cluster_status;`

#### Adding new administrator user

* Add a new/fresh user, say user ‘test’ and password ‘test’

`$ sudo rabbitmqctl add_user test test`

* Give administrative access to the new user

`$ sudo rabbitmqctl set_user_tags test administrator`

* Set permission to newly created user

`$ sudo rabbitmqctl set_permissions -p / test ".*" ".*" ".*"`

#### HA Queues

* To make all queues HA run this command at master. With this policy enabled RabbitMQ sync all queues to all nodes.

`$ sudo rabbitmqctl set_policy ha-all "" '{"ha-mode":"all","ha-sync-mode":"automatic"}'`
