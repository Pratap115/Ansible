# AWS EC2 Configuration Automation with Ansible

Discovers running EC2 instances tagged `AnsibleManaged=true` and configures baseline packages using the AWS EC2 dynamic inventory plugin.

## Design
- GitHub Actions authenticates to AWS through OIDC (no long-lived AWS access keys).
- `amazon.aws.aws_ec2` discovers instances in the selected region.
- SSH connects to each instance using a dedicated deployment key.
- Only explicitly tagged instances are selected.

## AWS prerequisites
1. Create an IAM role trusted by your GitHub repository's OIDC identity.
2. Restrict the trust policy to your exact GitHub organization/user, repository and branch/environment.
3. Grant the role only `ec2:DescribeInstances` for discovery. Add other permissions only if later automation needs them.
4. Ensure selected EC2 instances have public IPs reachable from the runner, or use a self-hosted runner inside the VPC.
5. Permit SSH only from an approved source. GitHub-hosted runner IP ranges change; a self-hosted runner or SSM-based connection is preferable for production.
6. Tag intended instances: `AnsibleManaged=true`. Optionally add `Environment=dev` and `Role=web`.

## GitHub configuration
Create GitHub Environment `aws-dev`:
- Secret `AWS_ROLE_TO_ASSUME`: ARN of the OIDC IAM role.
- Secret `EC2_SSH_PRIVATE_KEY`: SSH private key.
- Secret `EC2_SSH_KNOWN_HOSTS`: verified known_hosts entries.
- Variable `AWS_REGION`: for example `ap-south-1`.

The instance login user is set to `ubuntu`. Change `group_vars/all.yml` for Amazon Linux (`ec2-user`) or other AMIs. Ensure the public key corresponding to the private key is installed on the EC2 instances.

## Deploy
Actions > Configure AWS EC2 with Ansible > Run workflow.

## Verify
Review the job log's inventory graph and Ansible recap. Confirm the intended tagged hosts only were selected before enabling any broader automation.

## Security
Do not use AWS access keys in GitHub secrets when OIDC is available. Do not grant `ec2:*`. The sample only discovers and configures existing instances; it does not create or terminate EC2 resources.
