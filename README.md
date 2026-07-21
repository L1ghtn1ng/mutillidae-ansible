# Mutillidae Ansible

Deploy a pinned OWASP Mutillidae II release to a dedicated Ubuntu 24.04 LTS
host. The playbook installs Apache, PHP 8.3, and MariaDB, generates a self-signed
TLS certificate, and serves Mutillidae over HTTP and HTTPS.

> [!CAUTION]
> Mutillidae is intentionally vulnerable. Use an isolated lab VM on a private
> network. Do not expose the host to the public internet or deploy it on a
> machine that contains sensitive data.

## Requirements

- An Ubuntu 24.04 LTS target with an `ansible` user that has passwordless sudo
- SSH key authentication from the controller to the target
- [uv](https://docs.astral.sh/uv/getting-started/installation/) 0.11.x on the
  controller

Python and Ansible are managed by uv. The project pins `ansible-core` and all
Ansible collections; no global Python packages are required.

## Deploy

Update the address in `hosts.yml`, then install the locked dependencies and
collections:

```shell
uv sync --locked
uv run ansible-galaxy collection install --requirements-file requirements.yml
```

Confirm SSH and privilege escalation work before changing the target:

```shell
uv run ansible mutillidae --module-name ansible.builtin.ping
```

Deploy Mutillidae:

```shell
uv run ansible-playbook main.yml
```

After the first deployment, check-mode is useful for previewing later changes:

```shell
uv run ansible-playbook main.yml --check --diff
```

The application is limited by its upstream `.htaccess` to localhost and RFC1918
private networks. Add the target address to the controller's `/etc/hosts` if
you want to use its virtual-host names:

```text
192.168.1.50 mutillidae.localhost www.mutillidae.localhost cors.mutillidae.localhost
```

Replace `192.168.1.50` with the target's private address. The certificate is
self-signed, so browsers will display a certificate warning.

## Configuration

Deployment defaults live in `group_vars/all.yml`. The main settings are:

| Variable | Purpose |
| --- | --- |
| `mutillidae_source_version` | Exact upstream Git revision to deploy |
| `mutillidae_db_name` | MariaDB database name |
| `mutillidae_db_user` | Restricted application database user |
| `mutillidae_db_password` | Lab database password |
| `mutillidae_install_root` | Git checkout location |
| `mutillidae_web_root` | Apache document root (`src/`) |

The checked-in database password is deliberately retained for this isolated
training lab. Override it through inventory or Ansible Vault if the lab's risk
model requires a secret.

## Validate

Run the same static checks used by CI:

```shell
uv lock --check
uv run ansible-lint
uv run ansible-playbook --syntax-check main.yml
```

## Molecule VM test

The default Molecule scenario creates an actual Ubuntu 24.04 cloud VM with
libvirt. On an Ubuntu controller, install the host dependencies:

```shell
sudo apt-get update
sudo apt-get install --yes \
  build-essential cloud-image-utils libvirt-clients libvirt-daemon-system \
  libvirt-dev pkg-config qemu-system-x86 qemu-utils virtinst
sudo systemctl enable --now libvirtd
sudo virsh net-start default
```

If the default network is already active, the last command can be skipped.
Then run:

```shell
uv sync --locked --group molecule
uv run ansible-galaxy collection install --requirements-file requirements.yml
sudo -v
uv run molecule test
```

Molecule uses KVM when `/dev/kvm` is available and falls back to QEMU software
emulation otherwise. The fallback is substantially slower. The scenario checks
Ubuntu and PHP versions, the pinned Git revision, service health, permissions,
database access, HTTP/HTTPS responses, and playbook idempotence. It always
destroys the VM after the test.

GitHub Actions runs both the static checks and this VM scenario on
`ubuntu-24.04`. Nested virtualization availability on hosted runners can vary;
the QEMU fallback keeps the job functional when KVM is unavailable.
