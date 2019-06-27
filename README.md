# SSH key setup

Create two sets of ssh keys:

- One for connecting the the web and database server. With passphrase.
- One for connecting to github to pull / clone the repo (ie: deploy key). Without passphrase.

```bash
ssh-keygen -t rsa -b 4096 -C foo@bar.com
```

- Copy the public server key onto the servers (either automatically, eg: upon Digital Ocean droplet creation) or manually to `/home/{{ deploy_user }}/.ssh/authorized_keys`
- Save the public deploy key on github in the relevant repo as a deploy key.

# Local Setup

### Install dependencies and activate virtualenv

```bash
pipenv install
pipenv shell
```

### Edit `group_vars/all` inventory file

- Edit inventory file with required variables
- Encrypt the file:

```bash
ansible-vault encrypt group_vars/all
```

- Commit the changes (with the encrypted `all` inventory file) to source control
- For future editing of this file, run:

```bash
EDITOR=nano ansible-vault edit group_vars/all
```

### Login to servers once to add them to allowed hosts

```bash
ssh -i ~/.ssh/{{ ssh_key_name }} root@web_server_ip
ssh -i ~/.ssh/{{ ssh_key_name }} root@database_server_ip
```

### Add ssh identity locally so that passphrases don't get asked for

```bash
ssh-add ~/.ssh/{{ ssh_key_name }}
```

# Deployment

### Run `init_config.yml` to setup non root users and revoke root login

```bash
ansible-playbook ./init_config.yml --ask-vault-pass --private-key=~/.ssh/{{ ssh_key_name }} -i hosts
```

### Run `web_and_dabase.yml` to setup the servers

```bash
ansible-playbook ./web_and_database.yml --ask-vault-pass --private-key=~/.ssh/{{ ssh_key_name }} -i hosts
```

### To start the playbook from a specific task (skipping those before):
 ```bash
ansible-playbook ./web_and_dabase.yml --private-key=~/.ssh/{{ ssh_key_name }} -i hosts --start-at-task="clone or pull latest code"
```