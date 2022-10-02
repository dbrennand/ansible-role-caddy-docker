# Ansible Role: dbrennand.caddy_docker

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-908a85?logo=gitpod)](https://gitpod.io/#https://github.com/dbrennand/ansible-role-caddy-docker)
![Ansible-Lint](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/ansible-lint.yml/badge.svg)
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

Container image repositories, names and tags used to deploy Caddy as a container. The `*builder_*` variables are only used when `caddy_docker_plugins` is populated.

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

## Molecule Tests

To test the role, use Molecule: `molecule test`

### `systemd` with cgroups v2

Spent an afternoon working to get `systemd` working inside a container with cgroups v2.

With cgroups v1, the method for running `systemd` inside a container was: `--privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro` however, using this in Gitpod workspaces I observed the following error inside the container:

```
Failed to create /init.scope control group: Read-only file system
Failed to allocate manager object: Read-only file system
[!!!!!!] Failed to allocate manager object.
Exiting PID 1...
```

:sad:

I quickly realised that this does not work in Gitpod workspaces because it uses [cgroups v2](https://github.com/gitpod-io/gitpod/issues/9775).

After much Googling I found this [serverfault](https://serverfault.com/questions/1053187/systemd-fails-to-run-in-a-docker-container-when-using-cgroupv2-cgroupns-priva) thread which helped me find the solution.

The solution is to include `--cgroup-parent=docker.slice --cgroupns private` in the `docker run` command.

Example:

```bash
docker run -itd --rm \
  --privileged \
  --cgroup-parent=docker.slice --cgroupns private \
  --tmpfs /tmp --tmpfs /run --tmpfs /run/lock \
  geerlingguy/docker-debian10-ansible:latest
```

Once added, `systemd` starts as expected! Hurray!

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) for details.

## Authors & Contributors

[**dbrennand**](https://github.com/dbrennand) - *Author*
