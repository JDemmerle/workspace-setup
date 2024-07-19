# ------------------
# ANSIBLE-MAKEFILE v0.16.0
# Run ansible commands with ease
# ------------------
#
# Copyright (C) 2017 Paul(r)B.r
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU Affero General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Affero General Public License for more details.
#
# You should have received a copy of the GNU Affero General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.

SHELL:=/bin/bash

##
# VARIABLES
##
playbook   ?= setup
mkfile_dir ?= $(dir $(abspath $(lastword $(MAKEFILE_LIST))))
ifeq ("$(wildcard $(mkfile_dir)pass.sh)", "")
  opts     ?= $(args)
else # Handle vault password if any
  ifeq ("$(shell $(mkfile_dir)pass.sh 2> /dev/null)", "")
    opts     ?= $(args)
  else
    opts     ?= $(args) --vault-password-file=$(mkfile_dir)pass.sh
  endif
endif
ifneq ("$(limit)", "")
  opts     := $(opts) --limit="$(limit)"
endif
ifneq ("$(tag)", "")
  opts     := $(opts) --tag="$(tag)"
endif

##
# TASKS
##
.PHONY: install
install: ## make install [roles_path=roles/] # Install roles dependencies
	ansible-galaxy install --ignore-errors --role-file="requirements.yml" $(opts)

.PHONY: fetch-inventory
fetch-inventory: ## make fetch-inventory [provider=<ec2|gce...>] # Download dynamic inventory from Ansible's contrib
	@wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/$(provider).py
	@chmod +x $(provider).py

.PHONY: inventory
inventory: inventory-graph ## make inventory # LEGACY replaced by inventory-graph

.PHONY: inventory-graph
inventory-graph: ## make inventory-graph # Display the inventory as seen from Ansible
	@ansible-inventory --graph --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts)

.PHONY: inventory-list
inventory-list: ## make inventory-list # Display the inventory as seen from Ansible
	@ansible-inventory --list  --inventory inventories/netbox_inventory.yml -inventory inventories/clw/inventory $(opts)

.PHONY: lint
lint: ## make lint [playbook=setup] [args=<ansible-playbook arguments>] # Check syntax of a playbook
	@ansible-playbook --inventory-file inventories/netbox_inventory.yml --inventory-file inventories/clw/inventory --syntax-check $(opts) "$(playbook)"

.PHONY: debug
debug: mandatory-host-param ## make debug host=hostname [args=<ansible arguments>] # Debug a host's variable
	@ansible --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts) -m setup $(host)
	@ansible --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts) --module-name="debug" --args="var=hostvars[inventory_hostname]" $(host)

.PHONY: dry-run
dry-run: ## make dry-run [playbook=setup] [tag=<ansible tag>] [limit=<ansible host limit>] [args=<ansible-playbook arguments>] # Run a playbook in dry run mode
	@ansible-playbook --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory --diff --check $(opts) "$(playbook)"

.PHONY: run
run: ## make run [playbook=setup] [tag=<ansible tag>] [limit=<ansible host limit>] [args=<ansible-playbook arguments>] # Run a playbook
	@ansible-playbook --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory --diff $(opts) "$(playbook)"

group ?=all
.PHONY: list
list: ## make list [group=all] # List hosts inventory
	@ansible --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(group) --list-hosts

.PHONY: vault
vault: mandatory-file-param ## make vault file=/tmp/vault.yml [args=<ansible-vault arguments>] # Edit or create a vaulted file
	@[ -f "$(file)" ] && ansible-vault edit $(opts) "$(file)" || \
	ansible-vault create $(opts) "$(file)"

.PHONY: console
console: ## make console [args=<ansible-console arguments>] # Run an ansible console
	@ansible-console --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts)

group ?=all
.PHONY: facts
facts: ## make facts [group=all] [args=<ansible arguments>] # Gather facts from your hosts
	@ansible --module-name="setup"  --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts) --tree="out/" $(group)

.PHONY: tags
tags: ## make facts [args=<ansible arguments>] # List all tags available in the playbook
	@ansible-playbook --inventory inventories/netbox_inventory.yml --inventory inventories/clw/inventory $(opts) --list-tags "$(playbook)"

.PHONY: cmdb
cmdb: ## make cmdb # Create HTML inventory report
	@ansible-cmdb "out/" > list-servers.html

.PHONY: bootstrap
bootstrap: ## make bootstrap # Install ansible (Ubuntu only)
	@apt-get install -y software-properties-common && \
	apt-add-repository ppa:ansible/ansible && \
	apt-get update && \
	apt-get install -y ansible

.PHONY: mandatory-host-param mandatory-file-param
mandatory-host-param:
	@[ ! -z $(host) ]
mandatory-file-param:
	@[ ! -z $(file) ]

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

.DEFAULT_GOAL := help
