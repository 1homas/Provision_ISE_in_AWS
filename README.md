# Provision_ISE_in_AWS Playbook

1. Provision
    1. Create a single ISE Node which generates a 90-day Evaluation License pack
    1. Create a repository (S/FTP) for backups and hot/patching
    1. Apply Hot/Patches
1. Deploy
    1. System Certificate(s)
    1. Trusted Certificates(s)
    1. Roles
    1. Services
1. Configure
    1. Create repositories, users, identity stores, network devices, policies
1. Backup
    1. Initial configuration backup to the repository
    1. Keep certificates safe

## Quick Start

1. Clone this repository:  

    ```sh
    git clone https://github.com/1homas/Provision_ISE_in_AWS.git
    ```

1. Create your Python environment and install Ansible:  

    ```sh
    pip install --upgrade pip
    pipenv install --python 3.9
    pipenv install ansible 
    pipenv install boto boto3 botocore 
    pipenv install ciscoisesdk 
    pipenv install jmespath
    pipenv shell
    ansible-galaxy collection install cisco.ise --force
    ```

    If you have any problems installing Python or Ansible, see [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

1. Export your various keys, tokens, and credentials for your shell environment.

    ```bash
    # AWS IAM API Keys
    export AWS_REGION='us-west-1'
    export AWS_ACCESS_KEY='AKIAIOSF/EXAMPLE+KEY'
    export AWS_SECRET_KEY='wJalrXUtnFEMI/K7MDENG/bPxRfi/EXAMPLE+KEY'
    ```

    ```bash
    # ISE REST API Credentials
    export ISE_USERNAME='admin'
    export ISE_PASSWORD='ISEisC00L'
    export ISE_VERIFY=false
    export ISE_DEBUG=false
    ```

    Alternatively, keep your environment variables in `*.sh` files in a `.secrets` or similar folder in your home directory. Use `source {filename}` to load the environment variables from the files:

    ```bash
    source ~/.secrets/aws.sh
    source ~/.secrets/ise.sh
    ```

1. Verify your AWS regions are listed in the `inventory/aws_ec2.yaml` dynamic inventory file to ensure updates will be fast.  

1. Review the settings in `vars/main.yaml` and change them to match your desired cloud environment. :
    - `project_name`
    - `domain_name`
    - `aws.region` if your AWS region is not `us-west-1`
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
