# Ansible Role: dbrennand.caddy_docker

[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-908a85?logo=gitpod)](https://gitpod.io/#https://github.com/dbrennand/ansible-role-caddy-docker)
![Ansible-Lint](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/ansible-lint.yml/badge.svg)
![Molecule](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/molecule.yml/badge.svg)
![Ansible-Release](https://github.com/dbrennand/ansible-role-caddy-docker/actions/workflows/ansible-release.yml/badge.svg)

Ansible role to deploy [Caddy](https://caddyserver.com/) in a Docker container.

## Requirements

* [Docker](https://www.docker.com/)

* [Docker SDK for Python](https://docker-py.readthedocs.io/en/stable/)

* [`community.docker`](https://galaxy.ansible.com/community/docker) Ansible collection:

  ```bash
  ansible-galaxy collection install community.docker
  ```

## Role Variables

```yaml
caddy_docker_config_directory:
  path: ~/.config/caddy/
  # Optional
  # owner: owner
  # group: group
  # mode: 0755
caddy_docker_data_directory:
  path: ~/.local/share/caddy/
  # ...
```

Absolute path to Caddy config and data directories to be created. Attached to the container as bind mounts.

```yaml
caddy_docker_caddyfile: |-
  localhost
  respond "Hello, world!"
```

Contents of the [Caddyfile](https://caddyserver.com/docs/caddyfile) used to configure Caddy.

```yaml
caddy_docker_caddyfile_file:
  path: ~/.config/Caddyfile
  # ...
```

Absolute path to the Caddyfile to be created. Attached to the container as a bind mount.

```yaml
caddy_docker_image: caddy:2.6.2-alpine
caddy_docker_builder_image: caddy:2.6.2-builder
```

Container image repositories, names and tags used to deploy Caddy as a container. The `caddy_docker_builder_image` variable is only used when `caddy_docker_plugins` is populated.

```yaml
caddy_docker_builder_directory:
  path: /tmp/caddy-builder/
  # ...
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
caddy_docker_networks:
  - name: caddy
```

Names of the Docker networks to be created and attached to the Caddy container.

```yaml
caddy_docker_network_mode: default
```

Docker network mode to use for the Caddy container. The `caddy_docker_networks`, `caddy_docker_ports` and `caddy_docker_exposed_ports` variables have no affect when this variable is set to `host`.

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
- name: dbrennand.caddy_docker
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

### Example - Cloudflare DNS-01 Challenge

This example uses the [Cloudflare module for Caddy](https://github.com/caddy-dns/cloudflare) to obtain certificates (including wildcards) from Let's Encrypt for a [Cloudflare](https://www.cloudflare.com) managed domain. This is useful when you want to obtain certificates without opening port 80 (HTTP) to the internet.

You **must** generate an API token for Caddy to authenticate to the Cloudflare API and create a TXT record for the DNS-01 challenge.

1. Go to the [Cloudflare dashboard profile page](https://dash.cloudflare.com/profile).

2. On the left select **API Tokens** > **Create Token**.

3. Select the API token template named **Edit zone DNS**.

4. Modify the **Token name** to your liking.

5. Under *Permissions* select **+ Add more** and add the permission: `Zone / Zone / Read`.

6. Under *Zone Resources* include your zone: `Include / Specific zone / example.tld`.

7. *Optional* - Configure Client IP Address Filtering if desired.

8. Click **Continue to summary** > **Create Token**.

```yaml
- name: dbrennand.caddy_docker - Cloudflare
  hosts: all
  vars:
    # geerlingguy.pip role vars
    pip_install_packages:
      - name: docker
    # dbrennand.caddy_docker role vars
    caddy_docker_caddyfile: |-
      {
              email {$ACME_EMAIL}
      }

      # Cloudflare DNS-01 challenge
      (cloudflare) {
              tls {
                      dns cloudflare {$CLOUDFLARE_API_TOKEN}
              }
      }

      service.{$DOMAIN} {
              import cloudflare
              reverse_proxy container:port
      }
    caddy_docker_plugins:
      - github.com/caddy-dns/cloudflare
    caddy_docker_environment_variables:
      DOMAIN: domain.tld
      ACME_EMAIL: email@domain.tld
      CLOUDFLARE_API_TOKEN: token
  pre_tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: true
  roles:
    - geerlingguy.pip
    - geerlingguy.docker
    - dbrennand.caddy_docker
```

### Example - [Tailscale](https://tailscale.com/)

This example uses [artis3n/ansible-role-tailscale](https://github.com/artis3n/ansible-role-tailscale) to install Tailscale and configure Caddy to obtain a certificate from Let's Encrypt for your Tailscale node. You **must** have [MagicDNS](https://tailscale.com/kb/1081/magicdns/) and [HTTPS Certificate](https://tailscale.com/kb/1153/enabling-https/) features enabled for your Tailnet.

```yaml
- name: dbrennand.caddy_docker - Tailscale
  hosts: all
  vars:
    # geerlingguy.pip role vars
    pip_install_packages:
      - name: docker
    # artis3n.tailscale role vars
    tailscale_authkey: key
    # dbrennand.caddy_docker role vars
    caddy_docker_caddyfile: |-
      {
              email {$ACME_EMAIL}
      }

      # Tailscale
      (tailscale) {
              tls {
                      get_certificate tailscale
              }
      }

      node.{$TAILNET} {
              import tailscale
              reverse_proxy container:port
      }
    caddy_docker_extra_volumes:
      - /var/run/tailscale/tailscaled.sock:/var/run/tailscale/tailscaled.sock
    caddy_docker_environment_variables:
      ACME_EMAIL: email@domain.tld
      TAILNET: domain-alias.ts.net
  pre_tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: true
  roles:
    - geerlingguy.pip
    - geerlingguy.docker
    - artis3n.tailscale
    - dbrennand.caddy_docker
```

## Molecule Tests 🧪

To test the role, use molecule: `molecule test`

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) for details.

## Authors & Contributors

[**dbrennand**](https://github.com/dbrennand) - *Author*
