# VM Monitor: Automated EC2 Health Reports with Ansible

Automatically discovers AWS EC2 instances, collects health metrics (CPU, memory, disk), and emails an HTML report. No manual SSH into each server.

## How it works

```
EC2 instances (tagged Environment=dev, region eu-north-1)
            |
            v
Ansible control node (Ubuntu EC2)
   reads inventory/aws_ec2.yaml  ->  discovers running instances by tag
            |
            v
collect_metrics.yaml / playbook.yaml
   connects over SSH  ->  gathers CPU, memory, disk from each host
            |
            v
send_report.yaml
   loads SMTP variables from group_vars/all.yaml
   renders templates/report_email_animated.html.j2 (Jinja2)
            |
            v
community.general.mail  ->  HTML health report arrives in your inbox
```

## Features

- Dynamic inventory: instances are found by AWS tag, so there are no hard-coded IPs
- Metrics collected from every instance in one run
- HTML email report built from a Jinja2 template
- SMTP settings kept in variables, outside the playbooks
- Optional Bash scripts for local checks and EC2 tagging

## Project structure

```
.
├── ansible.cfg                  # default inventory and SSH settings
├── collect_metrics.yaml         # collects metrics from the instances
├── playbook.yaml                # main playbook
├── send_report.yaml             # builds and emails the HTML report
├── group_vars/
│   └── all.yaml                 # SMTP variables 
├── inventory/
│   └── aws_ec2.yaml             # AWS EC2 dynamic inventory
└── templates/
    └── report_email_animated.html.j2   # HTML email template
```

## Prerequisites

- Ubuntu control node (for example an EC2 instance) with Python 3
- AWS EC2 instances tagged `Environment=dev`
- AWS credentials: an IAM role on the control node (recommended) or `aws configure`
  - Needs `ec2:multile number of ec2 instances to check metrics`
- SSH key that can log in to the target instances
- An SMTP account (used **App Password** for Gmail )

## Setup

**1. Install Ansible and dependencies**

```sh
sudo apt update
sudo apt install -y ansible python3-boto3 python3-botocore
```

**2. Set the AWS region** (use the region where your instances run)

```sh
aws configure set region eu-north-1
```

**3. Check the inventory**

```sh
ansible-inventory -i inventory/aws_ec2.yaml --graph
```

You should see your instances listed under `aws_ec2` and `env_dev`.

**4. Create the email variables**

Create `group_vars/all.yaml` (this file must be named `all.yaml` so it applies to `localhost` too):

```yaml
smtp_server: smtp.gmail.com
smtp_port: 587
smtp_user: your-address@gmail.com
smtp_password: "your-app-password"
mail_to: recipient@example.com
```

**Never commit real passwords.** Either:

## Usage

Syntax check:

```sh
ansible-playbook playbook.yaml --syntax-check
```

Collect metrics and send the report:

```sh
ansible-playbook playbook.yaml
ansible-playbook send_report.yaml
```

If the variables file is vault-encrypted, add `--ask-vault-pass`.

## Optional Bash helpers

- `rename.sh`: renames tagged EC2 instances sequentially (`target-server-01`, `-02`, ...) using the AWS CLI
- `copy-public-key.sh`: copies the public key of the master server to the authorised key file in the target servers


## Security notes

- Do not commit `.pem` keys, AWS credentials, SMTP passwords, or shell history
- Prefer an IAM role over access keys on EC2
- Restrict SSH (port 22) in security groups to your own IP

## Tech stack

AWS EC2, Ansible, Jinja2, Bash, AWS CLI, Git, Linux

## Implementation Ideas

- Schedule runs with cron or a CI pipeline
- Add alert thresholds (email only when CPU, memory, or disk is high)
- Add more metrics and a history of reports

## Author

**Rozalan**: learning DevOps in public. Feedback is welcome.
