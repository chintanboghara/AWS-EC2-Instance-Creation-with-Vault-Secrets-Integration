# AWS EC2 Instance Creation with Vault Secrets Integration

This Terraform configuration demonstrates how to use AWS and Vault providers to securely fetch secrets and create an AWS EC2 instance with those secrets embedded as tags.

## Configuration

### AWS Provider
This script uses the AWS provider to interact with your AWS account. Update the region in the configuration as needed:
```hcl
provider "aws" {
  region = "ap-south-1"
}
```

### Vault Provider
The Vault provider fetches secrets from a Vault server. Replace `<VAULT_ADDRESS>`, `<ROLE_ID>`, and `<SECRET_ID>` with your actual Vault server details and AppRole credentials:
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

### Secrets in Vault
The script fetches secrets from a KV v2 engine. Ensure you have a secret stored in the path specified (`kv/secret`) with keys like `username`. Adjust the path as necessary:
```hcl
data "vault_kv_secret_v2" "example" {
  mount = "kv"
  name  = "secret"
}
```

### EC2 Instance Configuration
Modify the `ami` value to use your preferred AMI ID and adjust other properties as needed:
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

### Viewing Secrets in AWS Console
Once the EC2 instance is created, you can view the secret embedded as a tag in the AWS Management Console:
1. Navigate to the **EC2 Dashboard**.
2. Select the created EC2 instance.
3. Go to the **Tags** tab.
4. Look for the tag with the key `Secret` and verify the value.

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
   Confirm the deployment when prompted.

5. **Verify the EC2 Instance**:
   - Log in to your AWS Management Console and navigate to the EC2 dashboard.
   - Verify that the instance is created and tagged correctly.

## Cleanup
To destroy the resources created by this configuration:
```bash
terraform destroy
```
