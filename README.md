# A Dockerized Consul Container

[![](https://badge.imagelayers.io/troyfontaine/consul-server:latest.svg)](https://imagelayers.io/?images=troyfontaine/consul-server:latest 'Get your own badge on imagelayers.io')

This project is a Docker container for [Consul](http://www.consul.io/) and is a fork of the [gliderlabs/docker-consul](https://github.com/gliderlabs/docker-consul/) project which was the evolution of the [progrium/consul](https://github.com/gliderlabs/docker-consul/tree/legacy) project.

The focus is primarily on providing both documentation and simple configuration to allow anyone working with Docker Swarm a quick way to get a Discovery Backend up and running.

## Getting the container

The container is very small (less than 50MB, based on [Alpine Linux](https://hub.docker.com/_/alpine/)) and available on Docker Hub:

```$ docker pull troyfontaine/consul-server```

## Using the container

### Taking Consul for a spin

If you just want to run a single instance of Consul Server to try out its functionality:

```$ docker run  -p 8500:8500 -h node1 troyfontaine/consul-server -server -bootstrap```

The [Web UI](http://www.consul.io/intro/getting-started/ui.html) is enabled by default (via the pre-configured server.json) and is accessible via the url `http://yourhost:8500/ui/`.

In the above example, we are exposing ports 8400 (RPC), 8500 (HTTP) and 8600 (DNS). To ensure proper configuration, ensure that you set a host name via the -h flag when using docker run. This is used to set the Consul Agent node name by using the containers host name. 


### Leveraging Ansible to Launch Consul Containers

Using the Docker module in Ansible to launch a consul container is simple.  The set up of Ansible is outside the scope of this guide, so please visit the [Ansible website](http://docs.ansible.com) for more information on set up and general usage.
```
- name: "Testing Consul with Ansible"
  hosts: test1
  tasks:
    - name: "Launch Consul Container"
      become: True
      docker:
        name: "consul_node1"
        hostname: "node1"
        restart_policy: always
        image: troyfontaine/consul-server
        state: started
        ports:
          - "8500:8500"
      command: "-server -advertise {{ ansible_eth0.ipv4.address }} -bootstrap"
```


### Networking Consul in an Amazon VPC


## Advanced Configurations


### Setting the Data Center

`datacenter` or `-dc=` are [configuration or command arguments respectively](https://www.consul.io/docs/agent/options.html#_dc) to instruct Consul to talk to local Consul servers.

```$ docker run -p 8500:8500 -h node1 troyfontaine/consul-server -server -dc=MyDatacenter -bootstrap```

### Using Consul in a High Availability Configuration

Consul in a HA configuration requires a minimum of 3 "servers" to elect a leader.  It also requires several additional ports and has a variety of additional options to protect inter-server communications.

In our example below, we have a cluster of 3 Consul Server Containers with each container running on a single Docker host.

First Host: IP 192.168.0.10

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node1 troyfontaine/consul-server -server -advertise=192.168.0.10 -bootstrap-expect=3```

Second Host: IP 192.168.0.11

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node2 troyfontaine/consul-server -server -advertise 192.168.0.11 -join 192.168.0.10```

Third Host: IP 192.168.0.12

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node3 troyfontaine/consul-server -server -advertise 192.168.0.12 -encrypt=qoeGiN6VQT2QUrqgQ68xuG== -join 192.168.0.10```

### Gossip Encryption (Or How to Encrypt Traffic Between Consul Nodes)

`encrypt` or `-encrypt=` are [configuration or command arguments respectively](https://www.consul.io/docs/agent/options.html#_encrypt) to tell Consul to encrypt the "Gossip" traffic between nodes.  For more information on how to use this setting [click here](https://www.consul.io/docs/agent/encryption.html).  Consul features a built-in encryption key generator-but you can also use a password generator that can create a 22 character password that is letters (upper and lowercase) and numbers followed by two equals signs (==).

In our example below, we have a cluster of 3 Consul Server Containers with each container running on a single Docker host.

First Host: IP 192.168.0.10

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node1 troyfontaine/consul-server -server -advertise=192.168.0.10 -encrypt=qoeGiN6VQT2QUrqgQ68xuG== -bootstrap-expect=3```

Second Host: IP 192.168.0.11

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node2 troyfontaine/consul-server -server -advertise 192.168.0.11 -encrypt=qoeGiN6VQT2QUrqgQ68xuG== -join 192.168.0.10```

Third Host: IP 192.168.0.12

```$ docker run -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302 -p 8302:8302/udp -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node3 troyfontaine/consul-server -server -advertise 192.168.0.12 -encrypt=qoeGiN6VQT2QUrqgQ68xuG== -join 192.168.0.10```

### Providing a Custom Configuration File

This is where the awesome power of Docker really shines.  If you want to use a more advanced configuration file rather than the one included in the container image, you can either make a new image or mount a directory on the host system in-place of the included /config/ directory.

```$ docker run -p 8500:8500 -v <local/containers/consul/config>:/config/ -h node1 troyfontaine/consul-server -server -bootstrap```

For the complete list of Consul configuration options, click [here](https://www.consul.io/docs/agent/options.html).

## License

MIT
