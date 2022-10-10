# Ansible Role: dbrennand.caddy_docker

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-908a85?logo=gitpod)](https://gitpod.io/#https://github.com/dbrennand/ansible-role-caddy-docker)
![Ansible-Lint](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/ansible-lint.yml/badge.svg)
![Molecule](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/molecule.yml/badge.svg)
![Ansible-Release](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/ansible-release.yml/badge.svg)

Ansible role to deploy [Caddy](https://caddyserver.com/) in a Docker container.

## Requirements

* [Docker](https://www.docker.com/)

* [Docker SDK for Python](https://docker-py.readthedocs.io/en/stable/)

## Role Variables

```yaml
caddy_docker_config_path: /etc/caddy/config/
caddy_docker_data_path: /etc/caddy/data/
```

Absolute path to Caddy config and data directories to be created. Attached to the container as bind mounts.

```yaml
caddy_docker_caddyfile: |-
  localhost
  respond "Hello, world!"
```

Contents of the [Caddyfile](https://caddyserver.com/docs/caddyfile) used to configure Caddy.

```yaml
caddy_docker_caddyfile_path: /etc/caddy/Caddyfile
```

Absolute path to the Caddyfile to be created. Attached to the container as a bind mount.

```yaml
caddy_docker_image: caddy
caddy_docker_image_tag: 2.6.1-alpine
caddy_docker_builder_image: caddy
caddy_docker_builder_image_tag: 2.6.1-builder
```

Container image repositories, names and tags used to deploy Caddy as a container. The `caddy_docker_builder_*` variables are only used when `caddy_docker_plugins` is populated.

```yaml
caddy_docker_builder_directory: /etc/caddy/builder/
```

Absolute path for the directory used as the container build context. This variable is only used when `caddy_docker_plugins` is populated. You may want to override this variable if you bring your own dockerfile template and want to include files during the Caddy container's build process.

```yaml
caddy_docker_builder_template: dockerfile.j2
# Using the lookup plugin
# caddy_docker_builder_template: "{{ lookup('template', 'templates/dockerfile.custom.j2') }}"
```

Dockerfile template used to build the Caddy container. This variable is only used when `caddy_docker_plugins` is populated.

```yaml
caddy_docker_plugins: []
# Example
# caddy_docker_plugins:
#   - github.com/lucaslorentz/caddy-docker-proxy/plugin
#   - github.com/caddy-dns/cloudflare
```

List of plugins to include in the Caddy container.

```yaml
caddy_docker_ingress_network: caddy
```

Name of the ingress Docker network to be created and attached to the Caddy container.

```yaml
caddy_docker_command: caddy run --config /etc/caddy/Caddyfile --adapter caddyfile
```

Command for starting the Caddy container. You may want to override this when using plugins.

```yaml
caddy_docker_restart_policy: unless-stopped
```

Restart policy for the Caddy container.

```yaml
caddy_docker_ports:
  - 80:80
  - 443:443
  - "443:443/udp"
caddy_docker_exposed_ports: []
# Example
# caddy_docker_exposed_ports:
#   - 9000
```

Ports to expose on the Caddy container.

```yaml
caddy_docker_etc_hosts: {}
# Example
# caddy_docker_etc_hosts:
#   host.docker.internal: host-gateway
```

Host to IP mappings to place into the Caddy container's `/etc/hosts` file.

```yaml
caddy_docker_extra_volumes: []
# Example
# caddy_docker_extra_volumes:
#   - /site:/srv
```

Extra volumes to attach to the Caddy container.

```yaml
caddy_docker_environment_variables: {}
# Example
# caddy_docker_environment_variables:
#   DOMAIN: example.com
```

Environment variables to apply to the Caddy container.

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  vars:
    pip_install_packages:
      - name: docker
  pre_tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: true
  roles:
    - geerlingguy.pip
    - geerlingguy.docker
    - dbrennand.caddy_docker
```

## Molecule Tests 🧪

To test the role, use molecule: `molecule test`

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) for details.

## Authors & Contributors

[**dbrennand**](https://github.com/dbrennand) - *Author*
