# SSH key setup

Create two sets of ssh keys:

- One for connecting the the web and database server. With passphrase.
- One for connecting to github to pull / clone the repo (ie: deploy key). Without passphrase.

```bash
ssh-keygen -t rsa -b 4096 -C foo@bar.com
```

- Copy the public server key onto the servers (either automatically, eg: upon Digital Ocean droplet creation) or manually to `/home/{{ deploy_user }}/.ssh/authorized_keys`
- Save the public deploy key on github in the relevant repo as a deploy key.

Login to servers once to add them to allowed hosts:

```bash
ssh -i ~/.ssh/{{ ssh_key_name }} root@web_server_ip
ssh -i ~/.ssh/{{ ssh_key_name }} root@database_server_ip
```

Add ssh identity locally so that passphrases don't get asked for repeatedly: 

```bash
ssh-add ~/.ssh/{{ ssh_key_name }}
```

# Local Setup

## Install dependencies and activate virtualenv

```bash
pipenv install
pipenv shell
```

## Encrypted files

These files must remain encrypted when committing to source control:

- `group_vars/all`
- `roles/webserver/templates/django_settings.py`

The default password is `test`

For new projects, decrypt these files and then re-encrypt them with a new password:

```bash
ansible-vault decrypt group_vars/all
ansible-vault encrypt group_vars/all

ansible-vault decrypt roles/webserver/templates/django_settings.py
ansible-vault encrypt roles/webserver/templates/django_settings.py
```

Once the new password has been set, do not decrypt it. Rather edit it as follows which keeps the file encrypted between edits without risk of committing plain text to source control:

```bash
EDITOR=nano ansible-vault edit group_vars/all
```

## Deploy user

An encrypted password for `deploy_user_encrypted_password` can be created using:

```bash
mkpasswd --method=sha-512
```

## Python app dependencies

Check `roles/webserver/tasks/dependencies.yml` and update depending on whether `Poetry` or `Pipenv` is being used

# Deployment

## Run `init_config.yml` to setup non root users and revoke root login

```bash
ansible-playbook ./init_config.yml --ask-vault-pass --private-key=~/.ssh/{{ ssh_key_name }} -i hosts
```

## Run `web_and_dabase.yml` to setup the servers

```bash
ansible-playbook ./web_and_database.yml --ask-vault-pass --private-key=~/.ssh/{{ ssh_key_name }} -i hosts
```

## To start the playbook from a specific task (skipping those before):
 ```bash
 ansible-playbook ./web_and_database.yml --ask-vault-pass --private-key=~/.ssh/{{ ssh_key_name }} -i hosts --start-at-task="clone or pull latest code"
```