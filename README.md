https://access.redhat.com/solutions/6654601
https://github.com/ansible/instruqt/blob/devel/tracks/getting-started-servicenow-automation/06-servicenow-inventory/setup-controller
https://github.com/cloin/instruqt-snow

## Requisites

You will need a Red Hat Ansible subscription and an Automation Platform already installed.

## Configure Ansible Galaxy:

- Get an offline token from https://console.redhat.com/ansible/automation-hub/token
- Add to `/etc/ansible/ansible.cfg` the automation hub and the token

```
[galaxy]
server_list = automation_hub

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/content/published/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=$TOKEN
```

More info on the Automation Hub [documentation](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4/html/getting_started_with_automation_hub/configure-hub-primary#proc-configure-automation-hub-server-cli)

## Building the Execution Environment

Login on registry.redhat.io

```
$ podman login --log-level debug registry.redhat.io
```

Create the `Containerfile` into the context dir

```
$ cd ee
$ ansible-builder create -v 3
```

Build the EE

```
$ podman build -f context/Containerfile context --no-cache -t quay.io/pbertera/snow_ee:latest
```

Tag the image and push it to quay.io

```
$ podman login quay.io
$ podman push quay.io/snow_ee:latest
$ cd ..
```

## Configure the controller

Execute the Playbook

```
$ CONTROLLER="https://..."
$ USER="admin"
$ PASSWORD="*******"
$ SNOW_USER="xxxx"
$ SNOW_PASSWORD="*******"
$ SNOW_HOST0="https://xxxxx"
$ ansible-playbook -e controller_host="$CONTROLLER" \
                   -e controller_username="$USER" \
                   -e controller_password="$PASSWORD" \
                   -e snow_username="$SNOW_USER" \
                   -e snow_password="$SNOW_PASSWORD" \
                   -e snow_host="$SNOW_HOST" \
                   -e sync_project=false playbooks/controller/setup.yaml
```

At the end of the playbook execution you should have in your controller:
- A new execution environment pulling the image from `quay.io/pbertera/snow_ee:latest`
- A new credential type named `servicenow.itsm`
- The ServiceNow credential
- A new project pointing to this repository (depending on the `sync_project` variable value should be already sync)
- A new ServiceNow Inventory with a source sync
- A Job template executing the setup playbook (with all the variables in a survey)
- A dummy Job template listing all the hosts from the ServiceNow inventory

The repo URL and the EE image can be customized within the job survey.
