# ansible-role-victorialogs-docker

Ansible role for deploying victorialogs (vmalert and victoriametrics enabled by default but can be disabled) in docker container.

<img src="https://github.com/foi/ansible-role-victorialogs-docker/actions/workflows/ci.yml/badge.svg?branch=main">

Compatibility
--------------

python: 3.12, 3.13, 3.14
ansible: 12, 13

prerequisites
--------------

docker and docker-compose should be installed.

Install
--------------

`ansible-galaxy role install foi.victorialogs_docker`

or add it in `requirements.yml`

```
roles:
  - name: foi.victorialogs_docker
```

and run `ansible-galaxy install -r requirements.yml`

Role Variables
--------------
role defaults:
```yml
victorialogs_docker_version: 'v1.50.0'
victorialogs_docker_container_name: victorialogs
victorialogs_docker_root_folder: "/opt/{{ victorialogs_docker_container_name }}"
victorialogs_docker_root_folder_owner: root
victorialogs_docker_root_folder_group: root
victorialogs_docker_root_folder_mode: '0750'
victorialogs_docker_image: victoriametrics/victoria-logs
victorialogs_docker_compose_service_params:
  restart: unless-stopped
victorialogs_docker_compose_environment_params: {}
victorialogs_docker_compose_out_of_service_template:
  volumes:
    victorialogs-data: {}
    victoriametrics-data: {}
victorialogs_docker_compose_ports:
  - "9428:9428"
victorialogs_docker_compose_volumes:
  - "victorialogs-data:/victoria-logs-data"
victorialogs_docker_compose_retention: 1y
victorialogs_docker_compose_commands:
  - "-retentionPeriod={{ victorialogs_docker_compose_retention }}"
victorialogs_docker_vmalert_enabled: true
victorialogs_docker_vmalert_container_name: vmalert
victorialogs_docker_vmalert_version: "v1.139.0"
victorialogs_docker_vmalert_image: victoriametrics/vmalert
# vmalert rules {{  }} braces should be written as [[ ]] because {{  }} is reserved for jinja
# templating
# for cleanup vmalert rules you can set VICTORIALOGS_DOCKER_VMALERT_CLEANUP_CONFIGS=1 environment variable
victorialogs_docker_compose_vmalert_rules: {}
victorialogs_docker_compose_vmalert_compose_service_params:
  restart: unless-stopped
victorialogs_docker_compose_vmalert_environment_params: {}
victorialogs_docker_compose_vmalert_ports:
  - "8880:8880"
victorialogs_docker_compose_vmalert_rules_path: "{{ victorialogs_docker_root_folder }}/rules"
victorialogs_docker_compose_vmalert_volumes:
  - "{{ victorialogs_docker_compose_vmalert_rules_path }}:/etc/alerts"
victorialogs_docker_compose_vmalert_notify_url: "http://changethaturl:9093"
victorialogs_docker_compose_vmalert_datasource_url: "http://{{ victorialogs_docker_container_name }}:9428"
victorialogs_docker_compose_vmalert_remote_read_url: "http://{{ victorialogs_docker_victoriametrics_container_name }}:8428"
victorialogs_docker_compose_vmalert_remote_write_url: "http://{{ victorialogs_docker_victoriametrics_container_name }}:8428"
victorialogs_docker_compose_vmalert_service_params:
  restart: unless-stopped
victorialogs_docker_compose_vmalert_commands:
  - "--rule.defaultRuleType=vlogs"
  - "--datasource.url={{ victorialogs_docker_compose_vmalert_datasource_url }}"
  - "--remoteRead.url={{ victorialogs_docker_compose_vmalert_remote_read_url }}"
  - "--remoteWrite.url={{ victorialogs_docker_compose_vmalert_remote_write_url }}"
  - "--notifier.url={{ victorialogs_docker_compose_vmalert_notify_url }}"
  - "--rule='/etc/alerts/*.yml'"
victorialogs_docker_victoriametrics_enabled: true
victorialogs_docker_victoriametrics_version: "v1.139.0"
victorialogs_docker_victoriametrics_image: victoriametrics/victoria-metrics
victorialogs_docker_victoriametrics_container_name: victoriametrics
victorialogs_docker_compose_victoriametrics_ports:
  - "8428:8428"
victorialogs_docker_compose_victoriametrics_volumes:
  - "victoriametrics-data:/victoria-metrics-data"
victorialogs_docker_compose_victoriametrics_commands: {}
victorialogs_docker_compose_victoriametrics_service_params:
  restart: unless-stopped
victorialogs_docker_compose_victoriametrics_environment_params: {}
victorialogs_docker_compose_template: |
  services:
    {{ victorialogs_docker_container_name }}:
      image: {{ victorialogs_docker_image }}:{{ victorialogs_docker_version }}
      container_name: {{ victorialogs_docker_container_name }}
  {% for k, v in victorialogs_docker_compose_service_params.items() %}
      {{ k }}: {{ v }}
  {% endfor %}
  {% if victorialogs_docker_compose_environment_params.keys() | length > 0 %}
      environment:
  {% for k, v in victorialogs_docker_compose_environment_params.items() %}
        {{ k }}: {{ v }}
  {% endfor %}
  {% endif %}
      volumes:
  {% for volume in victorialogs_docker_compose_volumes %}
        - {{ volume }}
  {% endfor %}
  {% if victorialogs_docker_compose_ports | length > 0 %}
      ports:
  {% for port in victorialogs_docker_compose_ports %}
        - {{ port }}
  {% endfor %}
  {% endif %}
      command:
  {% for cmd in victorialogs_docker_compose_commands %}
        - {{ cmd }}
  {% endfor %}

  {% if victorialogs_docker_vmalert_enabled %}
    {{ victorialogs_docker_vmalert_container_name }}:
      image: {{ victorialogs_docker_vmalert_image }}:{{ victorialogs_docker_vmalert_version }}
      container_name: {{ victorialogs_docker_vmalert_container_name }}
  {% for k, v in victorialogs_docker_compose_vmalert_service_params.items() %}
      {{ k }}: {{ v }}
  {% endfor %}
      depends_on:
        - "{{ victorialogs_docker_container_name }}"
  {% if victorialogs_docker_victoriametrics_enabled %}
        - "{{ victorialogs_docker_victoriametrics_container_name }}"
  {% endif %}
  {% if victorialogs_docker_compose_vmalert_environment_params.keys() | length > 0 %}
      environment:
  {% for k, v in victorialogs_docker_compose_vmalert_environment_params.items() %}
        {{ k }}: {{ v }}
  {% endfor %}
  {% endif %}
      volumes:
  {% for volume in victorialogs_docker_compose_vmalert_volumes %}
        - {{ volume }}
  {% endfor %}
  {% if victorialogs_docker_compose_vmalert_ports | length > 0 %}
      ports:
  {% for port in victorialogs_docker_compose_vmalert_ports %}
        - {{ port }}
  {% endfor %}
  {% endif %}
  {% if victorialogs_docker_compose_vmalert_commands | length > 0 %}
      command:
  {% for cmd in victorialogs_docker_compose_vmalert_commands %}
        - "{{ cmd }}"
  {% endfor %}
  {% endif %}
  {% endif %}

  {% if victorialogs_docker_victoriametrics_enabled %}
    {{ victorialogs_docker_victoriametrics_container_name }}:
      image: {{ victorialogs_docker_victoriametrics_image }}:{{ victorialogs_docker_victoriametrics_version }}
      container_name: {{ victorialogs_docker_victoriametrics_container_name }}
  {% for k, v in victorialogs_docker_compose_victoriametrics_service_params.items() %}
      {{ k }}: {{ v }}
  {% endfor %}
      depends_on:
        - "{{ victorialogs_docker_container_name }}"
  {% if victorialogs_docker_compose_victoriametrics_environment_params.keys() | length > 0 %}
      environment:
  {% for k, v in victorialogs_docker_compose_victoriametrics_environment_params.items() %}
        {{ k }}: {{ v }}
  {% endfor %}
  {% endif %}
      volumes:
  {% for volume in victorialogs_docker_compose_victoriametrics_volumes %}
        - {{ volume }}
  {% endfor %}
  {% if victorialogs_docker_compose_victoriametrics_ports | length > 0 %}
      ports:
  {% for port in victorialogs_docker_compose_victoriametrics_ports %}
        - {{ port }}
  {% endfor %}
  {% endif %}
  {% if victorialogs_docker_compose_victoriametrics_commands | length > 0 %}
      command:
  {% for cmd in victorialogs_docker_compose_victoriametrics_commands %}
        - "{{ cmd }}"
  {% endfor %}
  {% endif %}
  {% endif %}
  {% if victorialogs_docker_compose_out_of_service_template is not none %}
  {{ victorialogs_docker_compose_out_of_service_template | to_nice_yaml(indent=2) }}
  {% endif %}
```

Example Playbook
----------------
```yml
# inventory
[servers]
victorialogs
# playbook.yml
- hosts: servers
  roles:
    - role: foi.victorialogs_docker
# host_vars/victorialogs.yml
# if you do not need extra root directives in docker compose.yml
victorialogs_docker_compose_out_of_service_template: null
# if you want to add some alert rules
victorialogs_docker_compose_vmalert_rules:
  timeouts: |
    groups:
      - name: timeouts
        type: vlogs
        interval: 1m
        rules:
          - alert: IOTimeout
            expr: |
              _time: 2m | "i/o timeout" | stats count() as cnt | cnt:> 1
            annotations:
              summary: "[[ $value ]] i/o timeouts appeared!"
            labels:
              severity: critical
              env: dev
```
License
-------

MIT
