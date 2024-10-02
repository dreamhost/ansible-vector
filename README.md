# Ansible Role: Vector

An Ansible role to install and configure [Vector](https://vector.dev/), supporting both templated and static configuration files. This role allows you to define simple Vector configurations using YAML format in Jinja2 templates, or more advanced configurations including vrl in file format.

## Table of Contents

- [Requirements](#requirements)
- [Role Variables](#role-variables)
- [Dependencies](#dependencies)
- [Example Playbooks](#example-playbooks)
  - [Using a Templated Configuration](#using-a-templated-configuration)
  - [Using a Static Configuration File](#using-a-static-configuration-file)
- [Configuration Structure](#configuration-structure)

## Requirements

- Ansible 2.9 or higher
- Supported Operating Systems:
  - Debian-based distributions (e.g., Ubuntu)

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Source template file for Vector configuration
vector_template: 'vector.yaml.j2'

# Source static configuration file (overrides template if defined)
vector_static_config_file: ''

# Destination path for the Vector configuration file
vector_config_file: '/etc/vector/vector.yaml'

# General configuration settings for Vector (template config only)
vector_general_config: ''

# Configuration for sources, transforms, and sinks (template config only)
vector_sources_config: {}
vector_transforms_config: {}
vector_sinks_config: {}
```

## Dependencies
None.

## Example Playbooks
### Using a Templated Configuration
By default, the role uses a templated configuration. You can define your Vector configuration using YAML in Jinja2 templates.

Template File (`templates/vector.yaml.j2`):

```yaml
{{ '{{ vector_general_config | default("") }}' }}
sources:
  {{ '{{ vector_sources_config | to_nice_yaml | indent(2) }}' }}
{% if vector_transforms_config is defined %}
transforms:
  {{ '{{ vector_transforms_config | to_nice_yaml | indent(2) }}' }}
{% endif %}
sinks:
  {{ '{{ vector_sinks_config | to_nice_yaml | indent(2) }}' }}
```

## Playbook:

```yaml
- hosts: all
  roles:
    - role: ansible-vector
      vars:
        vector_general_config: |
          data_dir: /var/lib/vector
        vector_sources_config:
          my_source:
            type: syslog
            address: "0.0.0.0:514"
        vector_transforms_config:
          my_transform:
            type: regex_parser
            inputs:
              - my_source
            regex: '(?P<timestamp>[^ ]+) (?P<level>[^ ]+) (?P<message>.+)'
        vector_sinks_config:
          my_sink:
            type: console
            inputs:
              - my_transform
            encoding:
              codec: json
```

## Using a Static Configuration File
To use a static configuration file, specify the path to your static config file.

```yaml
- hosts: all
  roles:
    - role: ansible-vector
      vars:
        vector_static_config_file: 'files/vector_static.yaml'
```
Place your static config file (vector_static.yaml) in the files directory of the role or provide an absolute path.

## Configuration Structure
The role allows you to define Vector configuration using YAML dictionaries for sources, transforms, and sinks. These configurations are then rendered into the configuration file using the provided template.

## Example Variables:

```yaml
vector_general_config: |
  data_dir: /var/lib/vector

vector_sources_config:
  my_source:
    type: syslog
    address: "0.0.0.0:514"

vector_transforms_config:
  my_transform:
    type: regex_parser
    inputs:
      - my_source
    regex: "(?P<timestamp>[^ ]+) (?P<level>[^ ]+) (?P<message>.+)"

vector_sinks_config:
  my_sink:
    type: console
    inputs:
      - my_transform
    encoding:
      codec: json
```

Author Information
This role was forked from dzervas/ansible-vector. Credit goes to the original author for the initial work.

