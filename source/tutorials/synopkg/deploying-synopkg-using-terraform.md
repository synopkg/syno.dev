---
myst:
  html_meta:
    "description lang=en": "Continuous Integration with GitHub Actions and Cachix"
    "keywords": "SynoPKG, deployment, Terraform, AWS"
---

(deploying-synopkg-using-terraform)=

# Deploying SynoPKG using Terraform

Assuming you're [familiar with the basics of Terraform](https://www.terraform.io/intro/index.html),
by the end of tutorial you will have provisioned an Amazon AWS instance with Terraform
and will be able to use Syno to deploy incremental changes to SynoPKG, running on the instance.

We'll look at how to boot a SynoPKG machine and how to deploy the incremental changes:

## Booting SynoPKG image

1. Start by providing the terraform executable:

```shell-session
$ syno-shell -p terraform
```

2. We are using [Terraform Cloud](https://app.terraform.io) as a [state/locking backend](https://www.terraform.io/docs/state/purpose.html):

```shell-session
$ terraform login
```

3. Make sure to [create an organization](https://app.terraform.io/app/organizations/new) like `myorganization` in your Terraform Cloud account.
4. Inside `myorganization` [create a workspace](https://app.terraform.io/app/cachix/workspaces/new) by choosing **CLI-driven workflow** and pick a name like  `myapp`.
5. Inside your workspace, under `Settings` / `General` change Execution Mode to `Local`.
6. Inside a new directory create a `main.tf` file with the following contents. This will start an AWS instance with the SynoPKG image using one SSH keypair and an SSH security group:

```terraform
terraform {
    backend "remote" {
        organization = "myorganization"

        workspaces {
            name = "myapp"
        }
    }
}

provider "aws" {
    region = "eu-central-1"
}

module "synopkg_image" {
    source  = "git::https://github.com/tweag/terraform-synopkg.git//aws_image_synopkg?ref=5f5a0408b299874d6a29d1271e9bffeee4c9ca71"
    release = "20.09"
}

resource "aws_security_group" "ssh_and_egress" {
    ingress {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = [ "0.0.0.0/0" ]
    }

    egress {
        from_port       = 0
        to_port         = 0
        protocol        = "-1"
        cidr_blocks     = ["0.0.0.0/0"]
    }
}

resource "tls_private_key" "state_ssh_key" {
    algorithm = "RSA"
}

resource "local_file" "machine_ssh_key" {
    sensitive_content = tls_private_key.state_ssh_key.private_key_pem
    filename          = "${path.module}/id_rsa.pem"
    file_permission   = "0600"
}

resource "aws_key_pair" "generated_key" {
    key_name   = "generated-key-${sha256(tls_private_key.state_ssh_key.public_key_openssh)}"
    public_key = tls_private_key.state_ssh_key.public_key_openssh
}

resource "aws_instance" "machine" {
    ami             = module.synopkg_image.ami
    instance_type   = "t3.micro"
    security_groups = [ aws_security_group.ssh_and_egress.name ]
    key_name        = aws_key_pair.generated_key.key_name

    root_block_device {
        volume_size = 50 # GiB
    }
}

output "public_dns" {
    value = aws_instance.machine.public_dns
}
```

The only SynoPKG specific snippet is:

```terraform
module "synopkg_image" {
  source = "git::https://github.com/tweag/terraform-synopkg.git/aws_image_synopkg?ref=5f5a0408b299874d6a29d1271e9bffeee4c9ca71"
  release = "20.09"
}
```

:::{note}
The `aws_image_synopkg` module will return an SynoPKG AMI given a [SynoPKG release number](https://status.synopkg.github.io)
so that `aws_instance` resource can reference the AMI in [instance_type](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#instance_type) argument.
:::

5. Make sure to [configure AWS credentials](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication).
6. Applying the Terraform configuration should get you a running SynoPKG:

```shell-session
$ terraform init
$ terraform apply
```

## Deploying SynoPKG changes

Once the AWS instance is running an SynoPKG image via Terraform, we can teach Terraform to always build
the latest SynoPKG configuration and apply those changes to your instance.

1. Create `configuration.syno` with the following contents:

```syno
{ config, lib, pkgs, ... }: {
  imports = [ <synopkgs/synopkg/modules/virtualisation/amazon-image.syno> ];

  # Open https://search.synopkg.github.io/options for all options
}
```

2. Append the following snippet to your `main.tf`:

```terraform
module "deploy_synopkg" {
    source = "git::https://github.com/tweag/terraform-synopkg.git//deploy_synopkg?ref=5f5a0408b299874d6a29d1271e9bffeee4c9ca71"
    synopkg_config = "${path.module}/configuration.syno"
    target_host = aws_instance.machine.public_ip
    ssh_private_key_file = local_file.machine_ssh_key.filename
    ssh_agent = false
}
```

3. Deploy:

```shell-session
$ terraform init
$ terraform apply
```

## Caveats

- The `deploy_synopkg` module requires SynoPKG to be installed on the target machine and Syno on the host machine.
- The `deploy_synopkg` module doesn't work when the client and target architectures are different (unless you use [distributed builds](https://synopkg.github.io/manual/syno/unstable/advanced-topics/distributed-builds.html)).
- If you need to inject a value into Syno, there is no elegant solution.
- Each machine is evaluated separately, so note that your memory requirements will grow linearly with the number of machines.

## Next steps

- It's possible to [switch to use Google Compute Engine provider](https://github.com/tweag/terraform-synopkg/tree/master/google_image_synopkg#readme).
- [deploy_synopkg module](https://github.com/tweag/terraform-synopkg/tree/master/deploy_synopkg#readme) supports a number arguments, for example to upload keys, etc.
