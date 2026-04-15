# Vault
Vault is the built in encryption store for Ansible. GPG (and security key based
GPG keys) can be used to encrypt ansible data, enabling ansible deployments
with security key touches. A makefile makes this process reproducible.

Assumes a [Yubikey with GPG][a] and [Environment][b] setup.

## Initial Setup

Create the initial directory structure and generate [SSH keys][c] if needed.

  Playbook Path       | Notes
 ---------------------|-------------
  scripts/Makefile    | Makefile to reproducibly create secret materials.
  scripts/vault_gpg   | GPG decrypt vault key script.
  scripts/vault_rekey | Rotation and re-encrypt vault key script.
  certs/gpg           | GPG material.
  certs/ssh           | SSH material.
  certs/ansible*      | Ansible vaulted SSH public key material.

### Generate a random vault password to use
``` bash
pwgen -n 71 -C | head -n1 | gpg --armor --recipient {GPGID} -e -o certs/gpg/ansible.gpg

# Re-key existing vault data with new key if needed.
grep -rl '^$ANSIBLE_VAULT.*' . | xargs -t ansible-vault rekey
```

### Create Vault Scripts
!!! abstract "vault_gpg"
    0755 {USER}:{USER}

    ``` bash
    #!/bin/sh
    #
    # Use from make.
    #
    # pwgen -n 71 -C | head -n1 | gpg --armor --recipient {GPG ID} -e -o ansible.gpg
    #
    # Reference:
    # * https://disjoint.ca/til/2016/12/14/encrypting-the-ansible-vault-passphrase-using-gpg/
    # * https://www.cloudsavvyit.com/3902/how-to-use-ansible-vault-to-store-secret-keys/

    gpg --batch --use-agent --decrypt "$(dirname $(dirname $(realpath -s $0)))/certs/gpg/ansible.gpg"
    ```

!!! abstract "vault_rekey"
    0755 {USER}:{USER}

    ``` bash
    #!/bin/sh
    #
    # Use from make. Helper script for rekeying GPG encrypted vaults. Use Makefile.
    #
    # Reference:
    # * https://disjoint.ca/til/2016/12/14/encrypting-the-ansible-vault-passphrase-using-gpg/
    # * https://www.cloudsavvyit.com/3902/how-to-use-ansible-vault-to-store-secret-keys/

    gpg --batch --use-agent --decrypt "$(dirname $(dirname $(realpath -s $0)))/certs/gpg/ansible.gpg.new"
    ```

### Update Ansible Environment
!!! abstract "ansible.env"
    0644 {USER}:{USER}

    ``` bash
    # Add the following to ansible.env to redirect vault authentication.
    # Direnv auto-loads from root directory regardless of entrypoint; pwd will
    # always resolve to root dir. core, core.pub are the SSH keys to use for
    # ansible user.
    export ANSIBLE_PRIVATE_KEY_FILE="$(pwd)/certs/ssh/core"
    export ANSIBLE_VAULT_PASSWORD_FILE="$(pwd)/scripts/vault_gpg"

    # Include JSON for vaulted files (e.g. any vaulted files).
    export ANSIBLE_YAML_FILENAME_EXT='.yml, .vault, .json'
    ```

### Create Makefile
!!! info "Makefiles are TAB sensitive"

!!! abstract "Makefile"
    0644 {USER}:{USER}

    ``` make
    ###############################################################################
    # Ansible Secrets
    ###############################################################################
    # direnv guarantees current virtual environment setup and  location. Make
    # always executed from current directory.

    # Base locations
    ROOT_DIR         = ..
    ENC_DIR          = $(ROOT_DIR)/certs
    GPG_DIR          = $(ENC_DIR)/gpg
    SSH_DIR          = $(ENC_DIR)/ssh
    SCRIPTS_DIR      = $(ROOT_DIR)/scripts

    # Ansible locations
    ANS_ENV          = $(ROOT_DIR)/ansible.env
    ANS_VENV         = $(ROOT_DIR)/.venv/bin/activate
    # This can be placed anywhere, it will be the vaulted authorized_keys.
    ANS_USER_VAULT   = $(ROOT_DIR)/group_vars/all/users/ansible/vault
    ANS_SSH_KEY      = $(ENC_DIR)/ansible_key.vault
    ANS_SSH_PUB      = $(ENC_DIR)/ansible_pub.vault

    # GPG (use your specific GPG keys!)
    GPG_ID           = user@example.com
    GPG_URL          = https://keys.openpgp.org/vks/v1/by-fingerprint/{GPG_ID}
    GPG_VAULT        = $(GPG_DIR)/ansible.gpg
    GPG_SCRIPT       = $(SCRIPTS_DIR)/vault_gpg
    GPG_SCRIPT_REKEY = $(SCRIPTS_DIR)/vault_gpg_rekey

    # SSH settings
    SSH_KEY_NAME     = core

    help:
    	@echo "USAGE:"
    	@echo "  🛈 SSH Key Management 🛈"
    	@echo
    	@echo "  make ssh_install"
    	@echo "      Decrypts and installs core ansible ssh keys for current user."
    	@echo
    	@echo "  make ssh_clean"
    	@echo "      Removes ansible ssh generated key material."
    	@echo
    	@echo "  make ssh_keys"
    	@echo "      Generate ansible SSH keys."
    	@echo
    	@echo "  make ssh_import"
    	@echo "      Import $(SSH_KEY_NAME) keys into current user SSH directory."
    	@echo "      Does NOT clobber existing keys."
    	@echo
    	@echo "  🛈 GPG Vault Key Management 🛈"
    	@echo
    	@echo "  make vault_create"
    	@echo "      Generate GPG encrypted vault keys using GPG_ID, which must be"
    	@echo "      imported first."
    	@echo
    	@echo "  make vault_rotate"
    	@echo "      Rotate vault keys. Entire files are automatically rotated."
    	@echo "      Per-variable encryption must be manually migrated."
    	@echo
    	@echo "  make vault_rotate_finish"
    	@echo "      After 'make vault_rotate', use this to remove old vault key and"
    	@echo "      store new one."
    	@echo
    	@echo "  🛈 Dependencies 🛈"
    	@echo
    	@echo "  make deps"
    	@echo "      Install required package dependencies."
    	@echo
    	@echo "      Assumes requires packages are installed. Attempts OS detection"
    	@echo "      and automatic [ssh, gnupg, pwgen] install."

    .PHONY: help Makefile

    sanity_check:
    	@echo -n "Are you sure? [y/N] " && read ans && [ $${ans:-N} = y ]

    deps:
    	OS_ID := $(shell grep "^ID=" /etc/os-release | cut -d'=' -f2)
    	ifneq ($(filter $(OS_ID),cachyos manjaro arch),)
    		@sudo pacman --sync --refresh --sysupgrade --noconfirm openssh gnupg pwgen
    		@sudo systemctl start pcscd
    	else
    		@sudo apt update && sudo apt install --yes ssh gnupg2 pwgen
    	endif

    ###############################################################################
    # SSH Key Management Options
    ###############################################################################
    # SSH Keys are stored encrypted in GPG vault, using the ansible configuration
    # file in $(GPG_DIR). This must be specified in addition to the virtual
    # environment for the ansible vault tool to correct find the vault settings for
    # GPG decryption.

    ssh_install:
    	@echo 'Decrypting and installing $(SSH_KEY_NAME) for ansible ssh authentication ...'
    	@mkdir -pv "$(SSH_DIR)"
    	@echo '🛆 Security key auth 🛆'
    	@export ANSIBLE_CONFIG="$(ANS_ENV)";. $(ANS_VENV); cd $(ROOT_DIR) && ansible-vault decrypt $(ANS_SSH_KEY) --output="../$(SSH_DIR)/$(SSH_KEY_NAME)"
    	@echo '🛆 Security key auth 🛆'
    	@export ANSIBLE_CONFIG="$(ANS_ENV)";. $(ANS_VENV); cd $(ROOT_DIR) && ansible-vault decrypt $(ANS_SSH_PUB) --output="../$(SSH_DIR)/$(SSH_KEY_NAME).pub"
    	@echo 'Done.'

    ssh_clean: sanity_check
    	@echo '🗘 Removing ansible ssh key material 🗘'
    	-@sudo rm -rfv $(SSH_DIR)/$(SSH_KEY_NAME)*
    	@echo 'Done.'

    ssh_keys:
    	@echo '🗘 Generating keys (do NOT create passwordless keys) 🗘'
    	@ssh-keygen -b 4096 -t rsa -f "$(SSH_DIR)/$(SSH_KEY_NAME).newkey"
    	@mv "$(SSH_DIR)/$(SSH_KEY_NAME).newkey" "$(SSH_DIR)/$(SSH_KEY_NAME)"
    	@mv "$(SSH_DIR)/$(SSH_KEY_NAME).newkey.pub" "$(SSH_DIR)/$(SSH_KEY_NAME).pub"
    	@echo 'Done.'
    	@echo '🛆 Updating ansible management keys (GPG auth required) 🛆'
    	-@rm -f $(ANS_USER_VAULT)
    	@echo -n "---\n# Autogenerated from make keys\n\nauthorized_keys: " > $(ANS_USER_VAULT)
    	@cat $(SSH_DIR)/$(SSH_KEY_NAME).pub >> $(ANS_USER_VAULT)
    	@. $(ANS_VENV); cd $(ROOT_DIR) && ansible-vault encrypt $(ANS_USER_VAULT)
    	@echo 'Done.'
    	@echo '🗘 Exporting & Encrypting keys to ansible vault 🗘'
    	@cp -f "$(SSH_DIR)/$(SSH_KEY_NAME)" $(ANS_SSH_KEY)
    	@cp -f "$(SSH_DIR)/$(SSH_KEY_NAME).pub" $(ANS_SSH_PUB)
    	@. $(ANS_VENV); cd $(ROOT_DIR) && ansible-vault encrypt $(ANS_SSH_KEY)
    	@. $(ANS_VENV); cd $(ROOT_DIR) && ansible-vault encrypt $(ANS_SSH_PUB)
    	@echo 'Done.'

    ssh_import:
    	@echo "🗘 Importing $(SSH_KEY_NAME) keys to current user 🗘"
    	@cp -vn "$(SSH_DIR)/$(SSH_KEY_NAME)*" ~/.ssh
    	@echo 'Done.'

    ###############################################################################
    # GPG Management Options
    ###############################################################################
    # Create GPG encrypted vault key. Do not store unencrypted key material in git.

    vault_install:
    	@echo '🗘 Stopping existing gpg-agents to prevent key generation failure 🗘'
    	-@sudo killall gpg-agent 2>/dev/null

    vault_rotate_finish:
    	@echo '🗘 Finishing vault key rotation 🗘'
    	@mv -v $(GPG_VAULT).new $(GPG_VAULT)
    	@echo 'Done.'

    vault_create: vault_install
    	@echo '🗘 Creating GPG encrypted vault keys 🗘'
    	@pwgen -n 71 -C | head -n1 | gpg --armor --recipient $(GPG_ID) -e -o $(GPG_VAULT)
    	@echo 'Done.'

    vault_rotate: vault_install
    	@echo '🗘 Rotating GPG encrypted vault keys 🗘'
    	@echo '🗘 Creating NEW GPG encrypted vault key 🗘'
    	@pwgen -n 71 -C | head -n1 | gpg --armor --recipient $(GPG_ID) -e -o $(GPG_VAULT).new
    	@. $(ANS_VENV); grep -rl '^$$ANSIBLE_VAULT.*' $(SSH_KEY_NAME) | xargs -t ansible-vault rekey --vault-password-file $(GPG_SCRIPT) --new-vault-password-file $(GPG_SCRIPT_REKEY)
    	@echo 'Done.'
    	@echo
    	@echo 'ONLY rekeyed whole files. Encrypted variabled needs manual rotation.'
    	@echo 'Suspected files with variable encryption:'
    	@echo
    	@grep -rl '[[:blank:]]$$ANSIBLE_VAULT.*' $(SSH_KEY_NAME)
    	@echo
    	@echo '🛆 Manually rotate suspected files if needed 🛆'
    	@echo
    	@echo '   make vault_rotate_finish'
    ```

### Vault Encrypt Ansible SSH Keys
These will be decrypted and used for SSH connections on connection. Place in
**certs/ansible.{key,pub}.vault**.

## Managing Keys
Ansible commands requiring authentication will prompt for key touch when
configured correctly. The Makefile is only needed for future management.

``` bash
cd scripts
make
```

## Reference[^1][^2]
[^1]: https://www.cloudsavvyit.com/3902/how-to-use-ansible-vault-to-store-secret-keys
[^2]: https://disjoint.ca/til/2016/12/14/encrypting-the-ansible-vault-passphrase-using-gpg

[a]: https://r-pufky.github.io/docs/app/gpg
[b]: https://r-pufky.github.io/ansible_docs/#environment-setup
[c]: https://r-pufky.github.io/docs/network/ssh/#generate-4096-bit-rsa-certificates-add-user-to-ssh-group
