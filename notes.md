# Ansible roles

# Ansible Vault
`Ansible Vault helps us to store our secrets and credentials so they will not be exposed to users. For the vault, we can also store passwords. All the secrets and credentials, everything, will be stored in this vault. This vault file is on the control node server. And it is encrypted with a password and every time you edit this vault.yaml file, you have to copy the encrypted message and update the vault.yaml which is stored in group_vars folder -> vault.yaml.`

# Ansible meta, handlers
`Meta is used for dependencies. For example, if you want to configure catalog and, in the catalog, you have mentioned this meta folder, then before configuring the catalog, these dependency tasks will executed, which are mentioned in meta. `

`Handlers are notifiers. For example, if you want to notify that a task should be executed when there is a change or update, you can mention them in the handlers folder. For example, if you made a change to the nginx configuration file, or if there is a change in the nginx configuration file, then these handlers will help us to notify that nginx should be restarted. `



# Ansible folder structure

