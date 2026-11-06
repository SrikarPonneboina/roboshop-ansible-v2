# Ansible Role Structure Reference

An Ansible role uses a standardized directory layout to organize tasks, variables, files, templates, and configurations into reusable modules.

## Directory Tree

```text
my_role/
├── tasks/
│   └── main.yml      # Main list of tasks to execute
├── handlers/
│   └── main.yml      # Handlers triggered by tasks (e.g., service restarts)
├── library/          # Modules and plugins (optional)
├── defaults/
│   └── main.yml      # Default, low-priority variables
├── vars/
│   └── main.yml      # High-priority variables for the role
├── files/            # Static files to transfer to managed nodes
├── templates/        # Jinja2 dynamic template files
├── meta/
│   └── main.yml      # Role metadata, author info, and dependencies
└── README.md         # Documentation for the role
```

## Core Directories Explained

* **`tasks/`**: Contains the main list of actions executed by the role (starting with `tasks/main.yml`). You can also include other task files from here.
* **`handlers/`**: Contains actions (such as restarting a web server) that run only when notified by a task using the `notify` keyword.
* **`defaults/`**: Holds default variables with the lowest precedence. Anyone using your role can easily override these values.
* **`vars/`**: Holds role-specific variables with a higher precedence that are not meant to be easily overridden.
* **`files/`**: Stores static files that need to be copied onto the target servers.
* **`templates/`**: Stores Jinja2 templates that render configuration files dynamically using variables.
* **`meta/`**: Stores metadata about the role, including minimum Ansible version, license, and dependencies on other roles.



# Ansible RoboShop v2 — Quick Reference & Concept Notes

---

## 1. Project Directory Layout

Here is the standard, production-ready project folder structure for RoboShop v2:

```text
ansible-roboshop-v2/
├── ansible.cfg              # Ansible configuration (inventory path, roles path, log path)
├── inventory/
│   ├── dev.ini              # Development environment servers
│   └── prod.ini             # Production environment servers
├── group_vars/              # Variables for server groups (MUST be named group_vars)
│   ├── all/
│   │   ├── vars.yml         # Non-sensitive global variables
│   │   └── vault.yml        # Encrypted secrets (Ansible Vault)
│   ├── frontend.yml         # Variables specific to frontend servers
│   └── database.yml         # Variables specific to database servers
├── host_vars/               # Variables for individual single hosts (MUST be named host_vars)
│   └── db-master.yml
├── roles/                   # Directory holding all custom Ansible roles
│   ├── common/
│   ├── mongodb/
│   ├── mysql/
│   ├── redis/
│   ├── rabbitmq/
│   ├── catalogue/
│   ├── user/
│   ├── cart/
│   ├── shipping/
│   ├── payment/
│   └── frontend/
└── site.yml                 # Main playbook that calls and runs all roles

Ansible RoboShop v2 — Quick Reference & Concept Notes

1. Project Directory Layout

Here is the standard, production-ready project folder structure for RoboShop v2:

ansible-roboshop-v2/
├── ansible.cfg              # Ansible configuration (inventory path, roles path, log path)
├── inventory/
│   ├── dev.ini              # Development environment servers
│   └── prod.ini             # Production environment servers
├── group_vars/              # Variables for server groups (MUST be named group_vars)
│   ├── all/
│   │   ├── vars.yml         # Non-sensitive global variables
│   │   └── vault.yml        # Encrypted secrets (Ansible Vault)
│   ├── frontend.yml         # Variables specific to frontend servers
│   └── database.yml         # Variables specific to database servers
├── host_vars/               # Variables for individual single hosts (MUST be named host_vars)
│   └── db-master.yml
├── roles/                   # Directory holding all custom Ansible roles
│   ├── common/
│   ├── mongodb/
│   ├── mysql/
│   ├── redis/
│   ├── rabbitmq/
│   ├── catalogue/
│   ├── user/
│   ├── cart/
│   ├── shipping/
│   ├── payment/
│   └── frontend/
└── site.yml                 # Main playbook that calls and runs all roles
2. Ansible Role Architecture

An Ansible Role is simply a neat, organized folder structure that breaks big playbooks into small, reusable chunks. Instead of writing 1,000 lines in one file, you split tasks, templates, variables, and handlers into dedicated folders.

Inside any role (e.g., roles/mongodb/), the standard structure looks like this:

roles/mongodb/
├── defaults/
│   └── main.yml        # Default variables (Lowest priority - easy to override)
├── vars/
│   └── main.yml        # Role-specific constants (High priority - hard to override)
├── tasks/
│   └── main.yml        # Step-by-step automation commands to execute
├── handlers/
│   └── main.yml        # Tasks that only run when triggered by something else
├── files/              # Plain static files (copied as-is, no changes)
├── templates/          # Dynamic Jinja2 templates (.j2 files with {{ variables }})
├── meta/
│   └── main.yml        # Metadata & role dependencies (who runs first)
└── tests/              # Test files for testing the role locally
    ├── inventory
    └── test.yml
3. Core Role Folder Definitions (Simple & Easy to Remember)

🔹 What Handlers Are Used For

In Simple Terms: "Do something ONLY when something changed."
Example: You update nginx.conf. You only want to restart Nginx if the configuration file actually changed. If nothing changed, don't waste time restarting the service.
How it works: A task in tasks/main.yml includes a line like notify: Restart Nginx. At the end of the playbook, Ansible runs the handler named Restart Nginx. Handlers run once at the very end to avoid unnecessary service restarts.
🔹 What Meta Is Used For (meta/main.yml)

In Simple Terms: "Role info and Prerequisites."
Example: Before setting up the catalogue application, you MUST run the common role to install basic tools and set up the roboshop user.
How it works: You list dependent roles inside meta/main.yml. Ansible automatically executes those prerequisite roles before executing the current role.
🔹 How Default Vars Work (defaults/main.yml)

In Simple Terms: "Fallback values that anyone can easily change."
Example: Setting mongodb_port: 27017 or app_version: "v4".
How it works: defaults has the lowest priority in Ansible. If someone passes a different port number in group_vars or from the terminal command line, Ansible immediately overwrites the default. It ensures your role doesn't crash if a variable is missing, but lets users change values easily.
🔹 How File Folders Work (files/)

In Simple Terms: "Copy file exact as-is without touching its content."
Example: Static SSL certificates, shell scripts, or raw fixed config files.
How it works: Used with the ansible.builtin.copy module. Ansible takes the file from roles/<role_name>/files/my_file.conf and pushes it straight to the remote server. No variable substitution or processing happens.
🔹 How Templates Work (templates/)

In Simple Terms: "Smart dynamic files that fill in the blanks."
Example: A config file where the database IP changes depending on whether you're deploying to dev or prod.
How it works: Files inside templates/ end with .j2 (Jinja2 format). You write placeholders like MONGO_URL="mongodb://{{ db_host }}:27017/catalogue". When used with ansible.builtin.template, Ansible replaces {{ db_host }} with the real IP at runtime before saving the file on the remote machine.
4. Variable Precedence (Which Variable Wins?)

Ansible has 22 levels of variable priority, but you only need to remember 4 main levels for daily work:

[ Lowest Priority ]  --> defaults/main.yml        (Role fallback values)
[ Medium Priority ]  --> group_vars / host_vars   (Environment values: dev/prod)
[ High Priority   ]  --> vars/main.yml            (Internal role constants)
[ Highest Priority]  --> CLI extra vars (-e)      (Command-line overrides)
💡 Defaults vs. Best Practices

Use defaults/main.yml when: Defining settings that users might want to change (ports, paths, versions, user names).
Use vars/main.yml when: Setting internal constants specific to that role that nobody should ever change from outside (e.g., specific package names or OS internal commands).
Best Practice Rule: Put environment-wide values (like DB IP addresses or passwords) inside group_vars/, keep role defaults in defaults/main.yml, and reserve -e for temporary terminal overrides.
5. Ansible Vault (Handling Secrets)

What You Need to Know About Ansible Vault

What is it? Ansible’s built-in encryption system using AES-256.
Why do we use it? To encrypt sensitive data (passwords, API keys, database credentials, SSH keys) so you can safely commit your Ansible code to public or private Git repositories without exposing secrets.
The 2 Encryption Modes

File-Level Encryption: Encrypts an entire YAML file. Anyone opening the file sees only scrambled code. Example: Encrypting group_vars/all/vault.yml.

Inline Variable Encryption: Encrypts just a single variable string inside a plain readable YAML file. Example:

app_name: "roboshop"
db_port: 27017
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663864383164393233306138613437346338303165313333333464303738313833333838383833
6. Easy-to-Remember Ansible Vault Commands

What you want to do	Command in Simple English	Description
Create a new secret file	ansible-vault create vault.yml	Opens a fresh encrypted file in an editor after asking for a password.
Encrypt an existing file	ansible-vault encrypt group_vars/all/vault.yml	Locks down an existing plain YAML file in-place.
Decrypt a file permanently	ansible-vault decrypt group_vars/all/vault.yml	Unlocks an encrypted file back into plain readable text.
View file without decrypting	ansible-vault view group_vars/all/vault.yml	Displays contents on screen temporarily without decrypting the file.
Edit an encrypted file	ansible-vault edit group_vars/all/vault.yml	Safely opens the file in an editor and re-encrypts it automatically on save.
Encrypt a single string (Inline)	ansible-vault encrypt_string 'MyPassword123' --name 'db_pass'	Gives you an encrypted string snippet to paste inside any YAML file.
Change the vault password	ansible-vault rekey group_vars/all/vault.yml	Changes the encryption password of a vault file.
🚀 Running Playbooks with Vault

# Option 1: Prompt for the password in terminal when running
ansible-playbook -i inventory/dev.ini site.yml --ask-vault-pass

# Option 2: Use a local password file (MUST be listed in .gitignore)
ansible-playbook -i inventory/dev.ini site.yml --vault-password-file ~/.ansible_vault_pass
7. Crucial Rules & Quick Tips to Remember

Folder Naming Convention: group_vars and host_vars folder names are mandatory and hardcoded in Ansible. Never rename them to something like my_vars.
connection: local: Used ONLY when running tasks on your local control machine (e.g., calling AWS API to spin up EC2). Omit it when targeting remote nodes over SSH.
wait: false: Tells cloud modules (like amazon.aws.ec2_instance) to send the API request to AWS and move to the next task instantly without hanging around for the server to finish booting.
Backgrounding Playbooks in Linux:
nohup ansible-playbook site.yml &> /path/to/logfile.log &
nohup: Keeps process running even if SSH drops.
&>: Redirects both standard output and error messages to logfile.
&: Pushes execution to the background.
Git Safety: NEVER commit plain passwords or password files (.vault_pass) to GitHub. Add .vault_pass and unencrypted secret files to your .gitignore.


















# Ansible Vault
`Ansible Vault helps us to store our secrets and credentials so they will not be exposed to users. For the vault, we can also store passwords. All the secrets and credentials, everything, will be stored in this vault. This vault file is on the control node server. And it is encrypted with a password and every time you edit this vault.yaml file, you have to copy the encrypted message and update the vault.yaml which is stored in group_vars folder -> vault.yaml.`

# Ansible meta, handlers
`Meta is used for dependencies. For example, if you want to configure catalog and, in the catalog, you have mentioned this meta folder, then before configuring the catalog, these dependency tasks will executed, which are mentioned in meta. `

`Handlers are notifiers. For example, if you want to notify that a task should be executed when there is a change or update, you can mention them in the handlers folder. For example, if you made a change to the nginx configuration file, or if there is a change in the nginx configuration file, then these handlers will help us to notify that nginx should be restarted. `



# Ansible folder structure

