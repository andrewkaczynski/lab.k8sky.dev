---
title: Proxmox VE
summary: Proxmox VE
authors:
    - Andrzej Kaczynski
date: 2024-01-05
---

## Introduction

I decided to use [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview) for my Protectli Virtualization Platform. The main reason was ease of setup and good support and documented deployments from Protectli. I'm sure it is going to meet all my home lab requirements. There tons of documents in the Internet with tutorials for Proxmox. That was another reason of choosing this OS software.

## Installation

Normally, the Proxmox VE installation is very trivial. It is based on Debian Linux and comes with really handy installer. You simply need to download the official ISO file from the [Proxmox website](https://www.proxmox.com/en/downloads) and prepare the USB stick. 

However, with my Protectli I had few challenges. Mainly because I wanted to use its 8GB eMMC Storage for the purpose of OS installation. During installation, I was able to select my eMMC storage however the installer was failing with an error:

```bash
Unable to get device for partition 1 on device /dev/mmcblk0
```

Luckily I found this article [Install Proxmox VE on eMMC](https://ibug.io/blog/2022/03/install-proxmox-ve-emmc/), which helped me address the issue. It seems Proxmox and its installer do not like **/dev/mmcblk*** devices, probably considering the eMMC as not good place for hosting the OS. 

## Post-Installation

There were a few things to be configured after the installation like:
- Configure necessary repositories (By default Proxmox uses the ones which requires subscription. I wanted to stay with OS only).
- Install basic packages (VIM)
- Update *GRUB* to enable PCI-Passthrough/IOMMU
- etc.

And of course the VMs or LXC containers. As I told at the beginning, the infrastructure I build must be IaC controller. Here the OpenTofu comes with the help.

###  OpenTofu

Yes! I'm going to use OpenTofu instead of Terraform. The idea of this project is to use only Open Source tools. Unfortunately, recent HashiCorp decision of changing the Terraform license to BSL is turning it out of Open Source communities.

There are few providers available for OpenTofu which can help with configuring the Proxmox. Unfortunately, they are not very well. They lack a lot of options and are usually limited only to few simple settings. I decided to let go with the following one:

There is a provider for Proxmox. Unfortunately, it is limited to provision only VMs or LXC containers. Not much, but better this than nothing. Its documentation describes how to start by creating the necessary Role, User, and API Key.

[Proxmox Provider for Terraform](https://registry.terraform.io/providers/bpg/proxmox/latest)

I'm going to use it with OpenTofu to provision necessary VMs for the purpose of my future applications and Kubernetes cluster, but also to configure the Proxmox node.

The code of my OpenTofu configuration can be found on my GitHub repo: [https://github.com/andrewkaczynski/homelab-tofu](https://github.com/andrewkaczynski/homelab-tofu).

In order to start using the Proxmox provider, the basic minimum is to configure the terraform block and set the required providers:

```bash
terraform {
    required_providers {
        proxmox = {
            source = "bpg/proxmox"
            version = "0.43.3"
        }
    }
}
```

And configure the necessary provider options:

```bash
provider "proxmox" {
    endpoint = "https://192.168.18.100:8006/"
    # I don't have trusted certificate installed on my proxmox node
    insecure = true
}
```

The credentials are passed thanks to the following two environment variables:

```bash
export PROXMOX_VE_USERNAME=root@pam
export PROXMOX_VE_PASSWORD=mysupersecretpassword
```

Let's initialize the OpenTofu project first.

```bash
$ tofu init

Initializing the backend...

Initializing provider plugins...
- Finding bpg/proxmox versions matching "0.43.0"...
- Installing bpg/proxmox v0.43.0...
- Installed bpg/proxmox v0.43.0 (signed, key ID DAA1958557A27403)

Providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://opentofu.org/docs/cli/plugins/signing/

OpenTofu has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that OpenTofu can guarantee to make the same selections by default when
you run "tofu init" in the future.

OpenTofu has been successfully initialized!

You may now begin working with OpenTofu. Try running "tofu plan" to see
any changes that are required for your infrastructure. All OpenTofu commands
should now work.

If you ever set or change modules or backend configuration for OpenTofu,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Then create the very first resource, e.g. **proxmox_virtual_environment_cluster_options**

```bash
resource "proxmox_virtual_environment_cluster_options" "options" {
  language                  = "en"
  keyboard                  = "pl"
  email_from                = "andrzej@kaczynski.dev"
}
```

Finally plan & apply the changes

```bash
$ tofu plan -out plan

OpenTofu used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

OpenTofu will perform the following actions:

  # proxmox_virtual_environment_cluster_options.options will be created
  + resource "proxmox_virtual_environment_cluster_options" "options" {
      + crs_ha      = (known after apply)
      + email_from  = "andrzej@kaczynski.dev"
      + id          = (known after apply)
      + keyboard    = "pl"
      + language    = "en"
      + mac_prefix  = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: plan

To perform exactly these actions, run the following command to apply:
    tofu apply "plan"

$ tofu apply plan
proxmox_virtual_environment_cluster_options.options: Creating...
proxmox_virtual_environment_cluster_options.options: Creation complete after 0s [id=cluster]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```

### Ansible

Because the available OpenTofu providers does not allow to configure everything I want, I need to use Ansible for the rest of the things. I'm going to use the official Provider for Ansible. This is going to be my first time when using it. I hope it will meet my expectations.

I've created my ansible role & playbook for manage this part. You can find the source code on my github repo [https://github.com/andrewkaczynski/homelab-ansible](https://github.com/andrewkaczynski/homelab-ansible).
