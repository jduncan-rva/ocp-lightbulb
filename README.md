# OCP-Lightbulb

## Purpose

ocp-lighbulb is a fork of the Ansible Lighbulb workshop tool (https://github.com/ansible/lightbulb). It has been modified to deploy a workshop environment for or demo environment for OpenShift.

By default each user for a workshop gets a 3-node OCP cluster that is fully deployed and ready to do work. The number of nodes can easily be scaled up for larger demo or even POC environments.

For demos, you can deploy an environment for a single user and use it to your heart's content.

## Configuration

To use ocp-lighbulb, you need to create two additional files. `users.yaml` is the list of users to provision a lab environment for. `extra_vars.yaml` are configurations that are specific to you and your environment.

### users.yaml

```yaml
users:
  - name: Bob Smith
    username: bsmith
    email: bsmith@email.com
  - name: Jane Doe
    username: jdoe
    email: jdoe@email.com
    ...
```

### extra_vars.yaml

```
workshop_lead: Your Name                             # Name for person running the lab. This will show up in the email
ec2_key_name: ansible-lab                            # SSH key in AWS to put in all the instances
ec2_region: us-west-1                                # region where the nodes will live
ec2_name_prefix: SOME_UUID_FOR_YOUR_WORKSHOP         # name prefix for all the VMs
ec2_vpc_id: vpc-9fe24af7                             # EC2 VPC ID in your region
ec2_vpc_subnet_id: subnet-90e24af8                   # EC2 subnet ID in your VPC
sendgrid_api_key: 'SENDGRID_API_KEY'                 # SendGrid API Key
instructor_email: 'Your Name <your@email.com>'       # address you want the emails to arrive from
admin_password: RedHat01                             # Set this to something better if you'd like. Defaults to 'LearnAnsible[two digit month][two digit year]', e.g., LearnAnsible0416
rhn_user: RHN_USER                                   # RHN username to regiser instances with
rhn_pass: rhn_pass                                   # RHN password to register instances with
ssh_key: /path/to/ssh/key                            # private key to put on OCP master. it will assume the public key to add to all hosts will be /the/same/path.pub
rhsm_pool_id: XXXX-XXXXXXXXXXX                       # Red Hat Pool ID to add your nodes to. This must include the OpenShift channels
```

## Amazon Setup

You will need an active EC2 account with the ability to spin up instances. That account will need to have a public key where you have a copy of the private key.

## Deploying ocp-lightbulb

```
$ git clone https://github.com/jduncan-rva/ocp-lightbulb.git
$ cd ocp-lightbulb
$ ansible-playbook --private-key ~/path/to/ec2/key -e @extra_vars.yml -e @users.yml aws_lab_setup/provision_lab.yml
```

## DNS

The default configuration, ocp-lightbulb uses the _nip.io_ domain for each cluster. Details about the nip.io domain are available at http://nip.io/.

### tl;dr:

> NIP.IO maps <anything>.<IP Address>.nip.io to the corresponding <IP Address>, even 127.0.0.1.nip.io maps to 127.0.0.1.    - http://nip.io

Each user's OCP cluster is configured with a subdomain configuration of <INFRA_NODE_IP>.nip.io. Any routes they create will have DNS records of <APP>-<NAMESPACE>.<INFRA_NODE_IP>.nip.io. This should resolve cleanly back to the IP address of the infra node where the router lives.

In short, DNS should _just work_, unless for some reason wherever you are blocks the nip.io domain.

## Users

By default, two users are created

* admin

This user has the password 'admin'. It is given the `cluster-admin` role, which means it is effectively all-powerful in the cluster.

* username

Each student also has a username created that maps to their username value in users.yml. This user is not associated with any roles by default at this point, but is added to the htpasswd file so they can log in.
