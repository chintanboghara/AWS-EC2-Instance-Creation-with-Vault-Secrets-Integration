# AWS EC2 Instance Creation with Vault Secrets Integration

This Terraform configuration demonstrates how to use the AWS and Vault providers to securely fetch secrets from a Vault server and create an AWS EC2 instance with those secrets embedded as tags.

## Prerequisites

- Terraform installed (version 0.13.0 or later recommended)
- An AWS account with permissions to create EC2 instances
- A Vault server with:
  - A KV v2 secrets engine mounted at `kv`
  - A secret stored at `kv/secret` with a key like `username`
  - An AppRole configured with policies to read the secret

## Configuration

### AWS Provider

The AWS provider interacts with your AWS account. The region is set to `ap-south-1`. Update it as needed:

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

Ensure your AWS credentials are configured (e.g., via environment variables or the AWS CLI).

### Vault Provider

The Vault provider fetches secrets from a Vault server. Replace `<VAULT_ADDRESS>`, `<ROLE_ID>`, and `<SECRET_ID>` with your Vault server URL and AppRole credentials:

```hcl
provider "vault" {
  address         = "<VAULT_ADDRESS>:8200"
  skip_child_token = true

  auth_login {
    path = "auth/approle/login"
    parameters = {
      role_id   = "<ROLE_ID>"
      secret_id = "<SECRET_ID>"
    }
  }
}
```

- `<VAULT_ADDRESS>`: The URL of your Vault server (e.g., `http://vault.example.com`)
- `<ROLE_ID>` and `<SECRET_ID>`: AppRole authentication credentials

The `skip_child_token = true` setting uses the same token for all requests.

### Secrets in Vault

Secrets are fetched from a KV v2 engine at the path `kv/secret`. Adjust the `mount` and `name` if your Vault setup differs:

```hcl
data "vault_kv_secret_v2" "example" {
  mount = "kv"
  name  = "secret"
}
```

Ensure the secret includes a key like `username` for use as a tag.

### EC2 Instance Configuration

The EC2 instance uses a specified AMI and instance type. Update the `ami` to a valid ID for your region, and adjust properties as needed:

```hcl
resource "aws_instance" "my_instance" {
  ami           = "ami-053b0d53c279acc90"
  instance_type = "t2.micro"

  tags = {
    Name   = "DemoEC2Instance"
    Secret = data.vault_kv_secret_v2.example.data["username"]
  }
}
```

- Verify the AMI ID matches your region.
- The `t2.micro` instance type is free-tier eligible but can be changed.

## Usage

1. **Initialize Terraform**:
   ```bash
   terraform init
   ```

2. **Validate the Configuration**:
   ```bash
   terraform validate
   ```

3. **Plan the Deployment**:
   ```bash
   terraform plan
   ```

4. **Apply the Configuration**:
   ```bash
   terraform apply
   ```
   Confirm when prompted.

5. **Verify the EC2 Instance**:
   - In the AWS Management Console, go to the **EC2 Dashboard**.
   - Select the instance and check the **Tags** tab for the `Secret` tag.
   - Alternatively, use the AWS CLI:
     ```bash
     aws ec2 describe-instances --instance-ids <instance_id> --query 'Reservations[].Instances[].Tags'
     ```

## Viewing Secrets in AWS Console

To confirm the secret is applied as a tag:
1. Navigate to the **EC2 Dashboard**.
2. Select the created instance.
3. Go to the **Tags** tab.
4. Check the `Secret` tag for the value from Vault.

## Cleanup

Remove all resources with:
```bash
terraform destroy
```

## Security Note

Tags are visible in the AWS console and not encrypted. Avoid using sensitive secrets in tags in production.

## Troubleshooting

- **Vault Errors**: Check AppRole credentials and policies.
- **AWS Errors**: Confirm credentials have EC2 permissions.
- **AMI Issues**: Ensure the AMI ID is valid for the region.
