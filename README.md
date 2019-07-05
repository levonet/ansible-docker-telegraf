Telegraf Role
==============
[![Build Status](https://travis-ci.org/levonet/ansible-docker-telegraf.svg?branch=master)](https://travis-ci.org/levonet/ansible-docker-telegraf)

Configure and start Telegraf in a docker container using the official image `telegraf:alpine`. 

Role Variables
--------------

- `docker_telegraf_config`: Body of configuration file.
- `docker_telegraf_name` (default: telegraf): This name used in name of home folder path and in name of container.
- `docker_telegraf_image` (default: telegraf:alpine)
- `docker_telegraf_periodic_restart` (default: no): Restart container at 4:15 AM everyday.
- `docker_telegraf_periodic_restart_time_hour` (default: 4)
- `docker_telegraf_periodic_restart_time_minute` (default: 15)
- `docker_telegraf_directory_volumes` (optional): Mount folders into docker
- `docker_telegraf_file_volumes` (optional): Mount files into docker
- `docker_telegraf_ports` (optional)
- `docker_telegraf_env` (optional)
- `docker_telegraf_network_mode` (default: host): Connect the container to a network.
- `docker_telegraf_networks` (default: []): List of networks the container belongs to.
- `docker_telegraf_log_driver` (default: json-file): Specify the logging driver.
- `docker_telegraf_log_options` (optional): Dictionary of options specific to the chosen log_driver. See [Configure logging drivers](https://docs.docker.com/engine/admin/logging/overview/) for details.

Dependencies
------------

- `docker`

Example Playbook
----------------

```yaml
- hosts: all
  become: yes
  become_method: sudo
  vars:
    influxdb_url: http://influxdb:8086
  roles:
    - role: levonet.docker_telegraf
      docker_telegraf_file_volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - /var/log/messages:/mnt/messages:ro
      docker_telegraf_config: |
        [global_tags]
          inventory = "{{ inventory_hostname }}"
        [agent]
          interval = "30s"
          round_interval = true
          metric_batch_size = 1000
          metric_buffer_limit = 10000
          collection_jitter = "0s"
          flush_interval = "10s"
          flush_jitter = "0s"
          precision = ""
          debug = false
          quiet = false
          logfile = ""
          hostname = ""
          omit_hostname = false
        [[outputs.influxdb]]
          urls = ["{{ influxdb_url }}"]
          database = "telegraf"
        [[inputs.cpu]]
          percpu = true
          totalcpu = true
          collect_cpu_time = false
          report_active = false
        [[inputs.disk]]
          ignore_fs = ["tmpfs", "devtmpfs", "devfs"]
        [[inputs.mem]]
        [[inputs.docker]]
         endpoint = "unix:///var/run/docker.sock"
        [[inputs.logparser]]
         files = ["/mnt/messages"]
         from_beginning = false
         [inputs.logparser.grok]
           patterns = ["%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:logsource:tag} %{PROG:program:tag}(?:\\[%{POSINT:pid:int}\\])?: %{GREEDYDATA:message}"]
           measurement = "messages"
```

License
-------

[MIT](https://opensource.org/licenses/MIT)

Author Information
------------------

This role was created by [Pavlo Bashynskyi](https://github.com/levonet)
