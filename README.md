# Provision_ISE_in_AWS Playbook
## Quick Start

1. Clone this repository:  

    ```sh
    git clone https://github.com/1homas/Provision_ISE_in_AWS.git
    ```

1. Create your Python environment and install Ansible:  

    ```sh
    python -m ensurepip --upgrade
    pip3   install --upgrade pipenv     # use a virtual development environment
    pipenv install --python 3.11        # use Python 3.9 or later
    pipenv install ansible 
    pipenv install boto boto3 botocore 
    pipenv install ciscoisesdk 
    pipenv install jmespath
    pipenv install paramiko             # ISE CLI
    pipenv shell
    ```

    > ⚠ Installing Ansible using Linux packages (`sudo apt install ansible`) may info in a much older version of Ansible being installed.
    > 💡 Installing Ansible with Python packages will get you the latest.
    > 💡 If you have any problems installing Python or Ansible, see [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

1. Export your various keys, tokens, and credentials for your shell environment.

    ```sh
    # AWS IAM API Keys
    export AWS_REGION='us-west-1'
    export AWS_ACCESS_KEY='AKIAIOSF/EXAMPLE+KEY'
    export AWS_SECRET_KEY='wJalrXUtnFEMI/K7MDENG/bPxRfi/EXAMPLE+KEY'
    ```

    ```sh
    export ISE_HOSTNAME=ise.securitydemo.net
    export ISE_USERNAME=admin
    export ISE_PASSWORD=ISEisC00L
    export ISE_VERIFY=False # optional, defaults to True
    # export ISE_VERSION=3.1_Patch_1 # optional, defaults to 3.1_Patch_1
    export ISE_WAIT_ON_RATE_LIMIT=True # optional, defaults to True
    export ISE_USES_API_GATEWAY=True # optional, defaults to True
    export ISE_DEBUG=False # optional, defaults to False
    export ISE_RADIUS_SECRET=ISEisC00L
    export ISE_TACACS_SECRET=ISEisC00L
    export DCLOUD_SESSION_ID=1234567
    ```

    > 💡 The cisco.ise Ansible modules will automatically use the `ISE_USERNAME`, `ISE_PASSWORD` and `ISE_VERIFY` variables so you do not need to reference them in your tasks!

    or you may edit and `source` these variables from a file in your `~/.secrets` directory :

    ```sh
    source ~/.secrets/aws.sh
    source ~/.secrets/ise.sh
    ```

1. Verify your AWS regions are listed in the `inventory/aws_ec2.yaml` dynamic inventory file to ensure updates will be fast.  

1. Review the settings in `vars/main.yaml` and change them to match your desired cloud environment. :
    - `project_name`
    - `domain_name`
    - `aws_region` if your AWS region is not `us-west-1`
    - AWS AMI identifiers 
    - your preferred network CIDR ranges in AWS
    - your instance types sizes for your ISE node(s)
    - your default password(s) or pre-shared keys

1. Edit the `provision.yaml` file and comment/uncomment the respective `ise_deployment_*.yaml` file for the deployment you want to provision.  

> ⚠ Be careful with the deployment and instance sizes... they may be *very* expensive to run if you are not actively using them!

## Provision

Provision your ISE instance(s) and wait for them to be available:

```sh
ansible-playbook -i inventory provision.yaml
```


The `provision.yaml` playbook creates the following :

- an AWS virtual private cloud (VPC) :
  - Internet Gateway
  - Public & Private Subnets
  - Public & Private Route Tables
- security group(s) for ISE
- ISE instance(s): software, CPU, RAM, storage, etc.
- DNS entries for each ISE node (assumes you have a domain in AWS)

There are potentially *many* more resources that could be created and applied: VPN gateways, certificates, repository, patches, etc. You are encouraged to provision these based on your requirements.  

You may also check availability with: 

```sh
ansible-playbook -i inventory wait_for_ise.yaml
```

## Deploy

Depending on the ISE deployment size and desired services, there are many more steps involved in making the provisioned *Standalone* ISE nodes above into an ISE deployment :

- Import Certificates
- Primary & Secondary Policy Administration Node (PAN) role election
- Primary & Secondary Monitoring & Troubleshooting (MNT) role election
- Policy Service Node (PSN) role election
  - Services configuration
  - Interfaces configuration
- Node Group creation and PSN assignment

You may add your tasks for these to the `deploy.yaml` playbook and run it :

1. Run the Ansible playbook:  

    ```sh
    ansible-playbook deploy.yaml
    ```

## Destroy

When you are done, you *should* terminate and remove all instances and associated resources to save money and prevent surprise bills from your cloud provider!

```sh
ansible-playbook destroy.yaml
```

## Resources

- [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for all platforms
- Documentation for Ansible collections:  
  - [amazon.aws](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)
  - [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/)
  - [ansible.netcommon](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/)
  - [azure.azcollection](https://docs.ansible.com/ansible/latest/collections/azure/azcollection/)
  - [cisco.ios](https://docs.ansible.com/ansible/latest/collections/cisco/ios/)
  - [cisco.ise](https://ciscoise.github.io/ansible-ise/main/plugins/)
  - [cisco.meraki](https://docs.ansible.com/ansible/latest/collections/cisco/meraki/)
  - [community.aws](https://docs.ansible.com/ansible/latest/collections/community/aws/)  
  - [community.azure](https://docs.ansible.com/ansible/latest/collections/community/azure/)
  - [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/)

## License

This repository is licensed under the [MIT License](https://mit-license.org/).
