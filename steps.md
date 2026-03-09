Step 0:
-------

    sudo docker pull opensdn/opensdn-tools
    sudo docker run --privileged --pid host --net host --name opensdn-tools -ti opensdn/opensdn-tools:latest
    git clone https://github.com/mkraposhin/opensdn-forwarder-basic-tutorial.git tut-rep
    tar cfz tut-rep.tgz tut-rep && sudo docker cp ./tut-rep.tgz opensdn-tools:/tut-rep.tgz && sudo docker exec -ti opensdn-tools tar xfz tut-rep.tgz
    sudo docker start opensdn-tools
    sudo modprobe vrouter
    sudo docker start cont1
    sudo docker start cont2
    sudo bash tut-rep/scripts/make-veth veth1 veth1c cont1 10.1.1.11/24
    sudo bash tut-rep/scripts/make-veth veth2 veth2c cont2 10.1.1.22/24
    sudo bash tut-rep/scripts/make-vif veth1
    sudo bash tut-rep/scripts/make-vif veth2

Step 1:
-------

1. Study OpenSDN data model
2. Study vRouter Agent and vRouter Forwarder communication


Step 2:
-------

Prepare network configuration using IF-MAP graph:
 - update global-config.xml, default-project.xml
 - update cont1.xml
 - update cont2.xml
 - update run-port-control-cont1
 - update run-port-control-cont2

Step 3:
-------

    git clone https://github.com/mkraposhin/opensdn-agent-basic-tutorial.git tut-agent
    sudo docker pull opensdn/opensdn-vrouter-agent
    sudo docker create --privileged --pid host --net host --name agent opensdn/opensdn-vrouter-agent:latest
    sudo docker cp tut-agent/scripts/entrypoint.sh agent:/
    sudo docker start agent

Step 4:
-------

Copy vrouter-port-control into $HOME:
    wget https://raw.githubusercontent.com/OpenSDN-io/tf-controller/refs/heads/master/src/vnsw/agent/port_ipc/vrouter-port-control

Step 5:
-------

Prepare Controller's ack message (controller-reply.xml)

Step 6:
-------

    sudo nc -4kl -s 10.0.2.15 -p 5269

Step 7:
-------

    sudo docker exec -ti agent bash

    # (inside the agent's container)
    source /actions.sh
    prepare_agent # (vhost0)
    create_agent_config
    /usr/bin/contrail-vrouter-agent


Step 8:
-------

after: <stream:stream from="opensdn-VirtualBox" to="network-control@contrailsystems.com" version="1.0" xml:lang="en" xmlns="" xmlns:stream="http://etherx.jabber.org/streams" >

Send hello via Controller (controller-reply.xml):

    <stream:stream from="network-control@contrailsystems.com" to="opensdn-VirtualBox" id="++123" version="1.0" xml:lang="en" xmlns="" xmlns:stream="http://etherx.jabber.org/streams" >


Step 9:
-------

- Send global-config.xml
- Send default-project.xml
- Send admin-project.xml
- Send cont1.xml
- Send cont2.xml


Step 10:
--------

    tut-agent/scripts/run-port-control-cont1
    tut-agent/scripts/run-port-control-cont2



Step 11:
--------

 - Check ports in the agent container: /var/lib/contrail/ports
 - Check VRFs table and interfaces
    - http://127.0.0.1:8085/Snh_VrfListReq?name=
    - http://127.0.0.1:8085/Snh_ItfReq

Step 12:
--------

Check connectivity

1. For ICMP connectivity, inside container 2, run:

    ping 10.1.1.11


2. For UDP connectivity:
    - inside container 1, run:
        nc -4ul -s 10.1.1.11 -p 12000
    - inside container 2, run:
        nc -4u 10.1.1.11 12000
    - send messages between containers
