# A Dockerized Consul Container

This project is a Docker container for [Consul](http://www.consul.io/) and is a fork of the gliderlabs/docker-consul project which was the evolution of the progrium/consul project.

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

### Using Consul in a High Availability Configuration


### Networking Consul in an Amazon VPC


### Leveraging Ansible to Launch Consul Containers
Using the Docker module in Ansible to launch a consul container is simple.  The set up of Ansible is outside the scope of this guide, so please visit the [Ansible website](http://docs.ansible.com) for more information on set up and general usage.
```
- name: "Testing Consul with Ansible"
  hosts: test1
  tasks:
    - name: "Docker Swarm Manager | Launch Discovery Backends"
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


### Providing a Custom Configuration File
This is where the awesome power of Docker really shines.  If you want to use a more advanced configuration file rather than the one included in the container image, you can either make a new image or mount a directory on the host system in-place of the included /config/ directory.

```$ docker run -p 8500:8500 -v <local/containers/consul/config>:/config/ -h node1 troyfontaine/consul-server -server -bootstrap```

For the complete list of Consul configuration options, click [here](https://www.consul.io/docs/agent/options.html).

## License

MIT
