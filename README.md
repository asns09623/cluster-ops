# cluster-ops

## Prerequisites
1. Ensure you have AWS CLI, Ansible, and required dependencies installed.
2. Configure AWS CLI with appropriate credentials:
   ```bash
   aws configure
   ```

## Steps to Create and Bootstrap the Cluster

### 1. Create SSH Key Pair
Generate an SSH key pair for the cluster:
```bash
aws ec2 create-key-pair --key-name kops-keypair --query 'KeyMaterial' --output text > kops-keypair.pem
chmod 400 kops-keypair.pem
```

Use 

### 2. Create the Cluster
Run the following command to create the cluster:
```bash
ansible-playbook cluster-create.yml
```

### 3. Bootstrap Addons

Update below values in bootstrap.yaml which you will get from terraform output.

- name: Setup public bastion and install addons from it
  hosts: localhost
  gather_facts: false

  vars:
    # ----- region / networking -----
    aws_region: ap-south-1
    vpc_id: "vpc-0a621bd1c28597dc7"
    public_subnets:
      - "subnet-0caadc9d32b00e2b6"
      - "subnet-0ef0b940a0871a01c"
      - "subnet-0560f9bf1ac2ef805"

    # ----- kOps / cluster -----
    state_store: "s3://example-oidc-bucket"
    cluster_name: "corp.dev.example.internal"
    hosted_zone_name: "corp.dev.example.internal"

    # ----- IRSA roles -----
    irsa_role_arns:
      autoscaler:   "arn:aws:iam::198549795675:role/kops-irsa-autoscaler"
      alb_ctrl:     "arn:aws:iam::198549795675:role/kops-irsa-alb_ctrl"
      external_dns: "arn:aws:iam::198549795675:role/kops-irsa-external_dns"
      cert_manager: "arn:aws:iam::198549795675:role/kops-irsa-cert_manager"
      argocd_repo:  "arn:aws:iam::198549795675:role/kops-irsa-argocd_repo"

environment:
        KOPS_STATE_STORE: "{{ state_store }}"
        AWS_ACCESS_KEY_ID: "" ##Update Keys
        AWS_SECRET_ACCESS_KEY: "" ##update secret key
Use the generated key pair to bootstrap addons:
```bash
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/kops-keypair.pem \
ansible-playbook addons-bootstrap.yml
```

## Additional Commands

### Upgrade the Cluster
To perform a rolling upgrade of the cluster:
```bash
ansible-playbook cluster-upgrade.yml
```

### Destroy the Cluster
To delete the cluster:
```bash
ansible-playbook cluster-destroy.yml
```

## Notes
- Ensure the `kops-keypair.pem` file is securely stored and has the correct permissions (`chmod 400`).
- Update the variables in the Ansible playbooks and group variables files as needed for your environment.
