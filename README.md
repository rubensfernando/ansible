# Brad's Bootstrapping & dotfiles Manager

Ansible playbook for bootstrapping macOS & Linux workstations, and managing dotfiles.

## Overview

This Ansible repo bootstraps and manages dotfiles for the following system types:

1. **macOS systems** `x86_64`, `arm64`
2. **Linux systems** `x86_64`
   1. **Fedora Workstation edition** (spins may work but are unsupported)
   2. **Fedora Server edition**
   3. **Pop!_OS Desktop LTS** (flavors may work but are unsupported)
   4. **Pop!_OS Server LTS**

The playbooks are designed with _no assumed prior knowledge of the system_, in contrast to how a server may be provisioned (e.g. images, PXE boot, etc). There are several steps to the process:

1. **Bootstrapping the OS:** (`docs/install`) Installs the necessary packages to check out the repository and run Ansible. On macOS this includes Homebrew (which also installs the Command Line Tools). For both, it includes Python and Pip. _This script requires `sudo` access on Linux only._

2. **Bootstrapping Ansible:** (`setup/site.yml`) A separate playbook that runs with its own Ansible configuration. It prompts the user to enter several configuration preferences for the host, and stores those settings in the Ansible inventory. _This playbook does not require `sudo` access._

3. **Bootstrapping the system:** (`playbooks/bootstrap.yml`) The bootstrap playbook is run to pull in additional software, configure the OS and applications, and setup the user environment (e.g. terminal, desktop, etc). _This playbook requires `sudo` access on both macOS and Linux._

4. **Configuring dotfiles**: (`playbooks/dotfiles.yml`) The dotfiles playbook is imported by the bootstrap playbook, and is meant to be run on its own periodically as user preferences and settings grow, adjust, or change. It can be run initially without bootstrapping, but it assumes all prerequisite packages are installed. _This playbook should never require `sudo` access._

### Vaulted secrets

No secrets are saved to the repository, so a new Ansible Vault passphrase is generated by the setup playbook. It's saved to `~/.ansible/vault` and should be recorded into your password manager of choice, and the file deleted.

There are currently four items that are encrypted with Vault:

1. The rsa SSH key
2. The ed25519 SSH key
3. GitHub personal access token
4. GitHub work access token

Post-bootstrap, the playbook `secrets.yml` will print out the unencrypted Vault values. This playbook can be run manually with `ansible-playbook playbooks/secrets.yml --ask-vault-password`.

## Running

```shell
curl -sO https://dotfiles.franklybrad.com/install
```

_Do not pipe `curl` into `sh` as Ansible won't run in interactive mode and will skip the setup prompts._

```text
sh install [-g git_branch] [-d]
  -g  Specify the git branch to run (default: main)
  -d  Run the dotfiles playbook only
```

### Pre-bootstrap tasks

* On macOS, log in to the App Store prior to running bootstrap, in order to install apps via `mas`
* Create a personal and/or work GitHub fine-grained token with the following permissions:
  * Repository permissions: All repositories
    * Contents: Read-only
  * Account permissions:
    * Git SSH keys: Read and write
    * SSH signing keys: Read and write

### Post-bootstrap tasks

* Record and delete the Vault passphrase (`~/.ansible/vault`)
* Record the SSH passphrases
* A full reboot
* To install Logi Options+ for macOS, run `open -a "~/Downloads/logioptionsplus_installer.app"`

## Housekeeping Tasks via Ansible

The following commands should be run from the top level of the repository.

* Update dotfiles: `ansible-playbook playbooks/dotfiles.yml --ask-vault-password`
* Regenerate `~/.ansible/inventory.yml`: `ANSIBLE_CONFIG=setup/setup.cfg ansible-playbook setup/site.yml`
* Update Ansible Galaxy collections: `ansible-playbook setup/collections.yml`
* Show vaulted secrets: `ansible-playbook playbooks/secrets.yml --ask-vault-password`

## Reusability

The playbooks are heavily personalized and customized to my needs, but in theory they can meet the needs of others through forking this repository and modifying the `group_vars` variables, and other various configs in `files` and `templates` to suit your needs.
