# Infrastructure as code for eCommerce Cloud Architecture on AWS (Multi Cloud AWS,GCP,Azure)

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)
<br/>

This repository contains Magento 2 Cloud Terraform infrastructure as code for AWS Public Cloud.

This infrastructure is the result of years of experience scaling Magento 1 and 2 in the cloud. It comes with the best cloud development practices baked in to save your time and money.

Leveraging your own AWS Account dramatically reduces your monthly spend vs. paying an expensive managed hosting provider (PaaS, SaaS).

This script is not limited to Magento deployments and can be used with any eCommerce/Web platform, eg. WordPress, WooCommerce, Drupal, Shopware 6, Shopify APP (Custum Private APP cloud), VueStorefront, Silyus, Oddo, ORO etc. It includes Magento in the name because it was designed for Magento at first. There are however projects using it to run Enterprise Java applications with auto scaling.


# Important!!!

Magento Software installation is out of the scope of this Project. This Repository is just an example of the AWS infrastructure provisioning for Magento using Terraform. Please refer to our another project to Install Magento 2 on Centos 8 or Amazon Linux 2 x86/ARM Linux: 

**Magento 2 Installation Automation (Centos 8.2, Amazon Linux 2 with ARM support) GitHub repository:**
[Magento installation Script] (https://github.com/nholuongut/magento-linux-installation).

Graviton 2 ARM instances are also supported. 


# Why Auto Scaling 

Increasing the number of PHP-FPM processes beyond the number of physical processor cores does not improve performance, rather is likely to degrade it, and can consume resources needlessly. Basic rule for the web is:

CPU(physical) = (Concurrent HTTP REquest * http_req_duration)

Be careful Intel CPUs are virtual and actual number of CPUs factor = 2; AWS Graviton2 ARM64 CPUs have factor 1 and are better for concurrent request processing. 
Intel CPUs have some advantages of 20-30% in some cases, however for magento (long heavy queries) physical cores are better. With higher traffic you need more CPUs.
It is rule for uncached pages.  

With Varnish/FPC it is the same. However Varnish has ~1ms response time and a single instance CPU can return 1000 caches pages per sec. To avoid unpredictable results with the cache invalidation, misses, uncached checkouts, cart, AJAXs, API the BEST practice is to measure performance without FPC. FPC is a bonus.


## AWS Magento 2 Cloud Features:
* True Horizontal Auto Scaling 
* Affordable (starting from ~300$ for us-west-2 region)
* MySQL RDS scalable Managed by Amazon, multi-az failover, vertical scaling with no downtime
* Compatible with RDS Aurora Cluster and Aurora Serverless
* EFS - Fully managed elastic NFS for media and configuration storage
* CloudFront CDN for static and media served from different origins S3 or Magento(EFS) as second origin 
* Automatically back up your code and databases (point-in-time snapshot) for easy restoration
* 99.9% Uptime, availability across multiple zones
* High security (Security groups, private infrastructure)
* Elastic(Static) IP and used for internet access for all EC2 instances through NAT (Network Address Translation).
* Bastion host to provide Secure Shell (SSH) access to the Magento web servers. 
* Appropriate security groups for each instance or function to restrict access to only necessary protocols and ports.
* Private Public Subnets - NAT gateway, Bastion server
* All servers and Database are securely hosted in private Network
* System and Software Update Patches
* DDoS Protection with AWS Shield
* PCI compliant infrastructure
* Redis cluster
* Amazon Elasticsearch Service - Elasticsearch at scale with zero down time with built-in Kibana
* Different Application Scaling Groups (ASG)
* Application Load Balancer(ALB) with SSL/TSL termination, SSL certificates management
* ALB Path-Based Routing, Host-Based Routing, Lambda functions as targets, HTTP header/method-based routing, Query string parameter-based routing 		
* Scaled Varnish ASG
* Dedicated Admin/Cron ASG
* You can easily add new autoscaling groups for your needs (Per WebSite/for Checkout requests/for API), just copy paste code 
* Possibility to run the same infrastructure on Production/Staging/Dev environment, different projects
* Automatic CI/CD (CodePipeline/CodeDeploy) deployments possible
* AWS CodeDeploy In-place deployment, Blue/green deployment from Git or S3, Redeploy or Roll Back
* Deploying from a Development Account to a Production Account
* Amazon Simple Email Service (Amazon SES) - cloud-based email sending service. Price $0.10 for 1K emails 
* Amazon CloudWatch - load all the metrics (CPU, RAM, Network) in your account for search, graphing, and alarms. Metric data is kept for 15 months.
* CloudWatch alarms that watche a single CloudWatch metric or the result of a math expression based on CloudWatch metrics and send SMS(Text) Notifications or Emails
* Simple and Step Scaling Policies - choose scaling metrics that trigger horizontal scaling
* Manual Scaling for Magento Auto Scaling Group (ASG)
* AWS Command Line Interface (CLI) - tool to manage your AWS services. You can control multiple AWS services from the command line and automate them through scripts.
* DynamoDB for logs, indexes, analytics
* Lambda functions as targets for a load balancer
* Elastic Container Registry (ECR) - fully-managed Docker container registry that makes it easy to store, manage, and deploy Docker container images!
* You can use Amazon Elastic Container Service (ECS) instead of ASG with Service Auto Scaling to adjust running containers desired count automatically.
* Awesome AWS documentation is Open Source and on GitHub

![Magento 2 AWS Infrastructure Cloud ](Magento2Cloud.png)

[Cloud Flat View](Magento2Cloud-Flat.png)

# Our Infrastructure

Infrastructure consists of multiple layers (autoscaling, alb, rds, security-group, vpc) where each layer is configured using one of the [Terraform AWS modules](https://github.com/nholuongut/terraform-aws-modules) with arguments specified in `terraform.tfvars` in layers directory.

Terraform uses this during the module installation step of `terraform init` to download the source code to a directory on local disk so that it can be used by other Terraform commands.

The [https://registry.terraform.io/](public Terraform registry) provides infrastructure modules for many infrastructure resources.

[Terragrunt](https://github.com/nholuongut/nholuong-terragrunt) is used to work with Terraform configurations which allows you to orchestrate dependent layers, update arguments dynamically and keep configurations. Define Terraform code once, no matter how many environments you have ([DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)).

# Minimal Magento Cloud Terraform Infrastructure 

![Magento Cloud Minimal Terraform Infrastructure](134946402-8a4ff61d-5def-448a-83dd-89eadecaa550.png)

The Minimal Magento Cloud infrastructure designed for small and extra large merchants. It can handle any load of up to 10,000 not cached requests per second(according to the internal test). Magento Commerce Cloud can’t handle even 100 simultaneous requests. Also, it dramatically reduces management overhead and cost.
After fixes in the Magento Fork Varnish is the redundant solution for 98% of the merchants and is not the best practice anymore.  

Sources of the small infrastructure located in the separate branch-> https://github.com/nholuongut/magento-2-cloud-on-aws-with-terraform/tree/minimal

# Magento 2 Multi Regional Infastructure Support 

We have a global scale-out model. All data updates (POST, DELETE request) are directed to the main data center region. All GET and CACHED requests (black lines) are routed to regional data centers. 

Geographically remote web servers add latency and degrade the shopping experience. Such mistakes can prove costly, resulting in lost customers, missed revenue, and reputational damage.

Route your traffic to your regional Magento Servers based on the user's location.
When you use geolocation routing, you can localize your web store and present some or all of your websites in the language of your users. You can also use geolocation routing to restrict access to the websites to only the locations you have distribution rights. Another use case is balancing load across endpoints.

## Pre-requirements

- [Terraform 0.12 or newer](https://www.terraform.io/)
- [Terragrunt 0.19 or newer](https://github.com/nholuongut/nholuong-terragrunt)
- [tfvars-annotations](https://github.com/nholuongut/tfvars-annotations) - Update values in terraform.tfvars using annotations
- Optional: [pre-commit hooks](http://pre-commit.com) to keep Terraform formatting and documentation up-to-date

# Install HomeBrew on Linux

Paste at a terminal prompt:
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
The installation script installs Homebrew to /home/linuxbrew/.linuxbrew using sudo if possible and, if not, in your home directory at ~/.linuxbrew. Homebrew does not use sudo after installation. Using /home/linuxbrew/.linuxbrew allows the use of more binary packages (bottles) than installing in your personal home directory.

The followig instructions will add Homebrew to your PATH and to your bash shell profile script (either ~/.profile on Debian/Ubuntu or ~/.bash_profile on CentOS/Fedora/RedHat).
```
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```
You’re done! Try installing a package:
```
brew install hello
```
If you’re using an older distribution of Linux, installing your first package will also install a recent version of glibc and gcc. Use `brew doctor` to troubleshoot common issues.

If you are using Mac you can install all dependencies using Homebrew:

    $ brew install terraform terragrunt pre-commit
    
## Manual install:

You can install Terragrunt manually by going to the [Releases page](https://github.com/nholuongut/nholuong-terragrunt/releases), downloading the binary for your OS, renaming it to terragrunt and adding it to your PATH.

# Install Terragrunt and Terraform Ubuntu Manually
```
sudo -s; ## run as a super user
    export TERRAFORM_VERSION=0.12.24 \
    && export TERRAGRUNT_VERSION=0.23.2 \
    && mkdir -p /ci/terraform_${TERRAFORM_VERSION} \
    && wget -nv -O /ci/terraform_${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
    && unzip -o /ci/terraform_${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip -d /usr/local/bin/ \
    && mkdir -p /ci/terragrunt-${TERRAGRUNT_VERSION}/ \
    && wget -nv -O /ci/terragrunt-${TERRAGRUNT_VERSION}/terragrunt https://github.com/nholuongut/nholuong-terragrunt/releases/download/v${TERRAGRUNT_VERSION}/terragrunt_linux_amd64 \
    && sudo chmod a+x /ci/terragrunt-${TERRAGRUNT_VERSION}/terragrunt \
    && cp /ci/terragrunt-${TERRAGRUNT_VERSION}/terragrunt /bin \
    && chmod a+x /bin/terragrunt \
    && rm -rf /ci \
    && exit
```
Test The Terragrunt/Terraform installation(Optional):
```
terragrunt -v;
terraform -v
```

## Instructions for use

Step 0. Terraform uses the SSH protocol to clone the modules. Configured SSH keys will be used automatically. Add your SSH key to github account. (https://help.github.com/en/enterprise/2.15/user/articles/adding-a-new-ssh-key-to-your-github-account)

Git+SSH is used because it works for both public and private repositories.

Step 1. Set credentials. By default, access credentials to AWS account should be set using environment variables:
```
     export AWS_DEFAULT_REGION=us-west-1 ## change it to your preferable AWS region
     export AWS_ACCESS_KEY_ID="..."
     export AWS_SECRET_ACCESS_KEY="..."
```
Alternatively, you can edit `common/main_providers.tf` and use another authentication mechanism as described in the [AWS provider documentation](https://www.terraform.io/docs/providers/aws/index.html#authentication).

The AWS provider offers a flexible means of providing credentials for authentication. The following methods are supported, in this order, and explained below:

Static credentials
Environment variables
Shared credentials/configuration file
CodeBuild, ECS, and EKS Roles
EC2 Instance Metadata Service (IMDS and IMDSv2)

Step 2. Once all arguments are set, run this command to create infrastructure in all layers in a single region:

    $ cd production
    $ terragrunt apply-all

Alternatively, you can create infrastructure in a single layer (eg, `autoscaling_3`):

    $ cd production/autoscaling_3
    $ terragrunt apply

See [official Terragrunt documentation](https://github.com/nholuongut/nholuong-terragrunt/blob/master/README.md) for all available commands and features.

If you are using newer version of the terragrunt you should use :

- **Region as a whole (slower&complete).** Run this command to create infrastructure in all layers in a single region:

```
$ cd ap-southeast-1
$ terragrunt run-all apply
```

- **As a single layer (faster&granular).** Run this command to create infrastructure in a single layer (eg, `magento_auto_scaling`):

```
$ cd ap-southeast-1/magento_auto_scaling
$ terragrunt apply
```

After the confirmation your infrastructure should be created.

## Destroy infrastructure

**destroy-all** (DEPRECATED: use run-all)
DEPRECATED: Use **run-all destroy** instead.

```
 terragrunt run-all destroy
```

Destroy a ‘stack’ by running ‘terragrunt destroy’ in each subfolder.


# Demo showing how it works (click on image)

<img alt="Magento AWS Cloud" src="Magento2Cloud-Flat.png" width="50%" height="50%">
</a>

Architecting your Magento platform to grow with your business can sometimes be a challenge. This video walks through the steps needed to take an out-of-the-box, single-node Magento implementation and turn it into a highly available, elastic, and robust deployment. This includes an end-to-end caching strategy that provides an efficient front-end cache (including populated shopping carts) using Varnish on Amazon EC2 as well as offloading the Magento caches to separate infrastructure such as [https://aws.amazon.com/elasticache/](Amazon ElastiCache). We also look at strategies to manage the Magento Media library outside of the application instances, including [https://aws.amazon.com/efs/](EFS shared storage solutions).


# Debug logging

If you set the TERRAGRUNT_DEBUG environment variable to “true”, the stack trace for any error will be printed to stdout when you run the app.

Additionally, newer features introduced in v0.19.0 (such as locals and dependency blocks) can output more verbose logging if you set the TG_LOG environment variable to debug.

Turn on debug when you need do troubleshooting.
```
# or if you run with terragrunt
TF_LOG=DEBUG terragrunt <command>
```

In the new versions of the terragrunt use:
```
terragrunt run-all apply --terragrunt-log-level debug --terragrunt-debug
```

Terragrunt and Terraform usually play well together in helping you write DRY, re-usable infrastructure. But how do we figure out what went wrong in the rare case that they don’t play well?

Terragrunt provides a way to configure logging level through the --terragrunt-log-level command flag. Additionally, Terragrunt provides --terragrunt-debug, that can be used to generate terragrunt-debug.tfvars.json.

For example you could use it like this to debug an apply that’s producing unexpected output:

```
$ terragrunt apply --terragrunt-log-level debug --terragrunt-debug
```

Running this command will do two things for you:

Output a file named terragrunt-debug.tfvars.json to your terragrunt working directory (the same one containing your terragrunt.hcl).
Print instructions on how to invoke terraform against the generated file to reproduce exactly the same terraform output as you saw when invoking terragrunt. This will help you to determine where the problem’s root cause lies.
Using those features is helpful when you want determine which of these three major areas is the root cause of your problem:

Misconfiguration of your infrastructure code.
 - An error in terragrunt.
 - An error in terraform.

# Clearing the Terragrunt cache

Terragrunt creates a .terragrunt-cache folder in the current working directory as its scratch directory. It downloads your remote Terraform configurations into this folder, runs your Terraform commands in this folder, and any modules and providers those commands download also get stored in this folder. You can safely delete this folder any time and Terragrunt will recreate it as necessary.

If you need to clean up a lot of these folders (e.g., after terragrunt apply-all), you can use the following commands on Mac and Linux:

Recursively find all the .terragrunt-cache folders that are children of the current folder:
```
find . -type d -name ".terragrunt-cache"
```
If you are ABSOLUTELY SURE you want to delete all the folders that come up in the previous command, you can recursively delete all of them as follows:
```
find . -type d -name ".terragrunt-cache" -prune -exec rm -rf {} \;
```
Also consider setting the TERRAGRUNT_DOWNLOAD environment variable if you wish to place the cache directories somewhere else.

# Destroy Terragrunt Magento Infrastructure 
```
terragrunt destroy-all 
```
Infrastructure managed by Terraform will be destroyed. This will ask for confirmation before destroying.

This command accepts all the arguments and flags that the apply command accepts, with the exception of a plan file argument.

If -auto-approve is set, then the destroy confirmation will not be shown.

The -target flag, instead of affecting "dependencies" will instead also destroy any resources that depend on the target(s) specified. For more information, see the [Targeting section of the terraform plan documentation](https://www.terraform.io/docs/commands/plan.html#resource-targeting).

The behavior of any terraform destroy command can be previewed at any time with an equivalent `terraform plan -destroy` command.


# Production & staging environments 

You can copy/paste folders to create new environments. Consider the following files structure, which defines three magento environments (prod, project-3 and stage) with the same infrastructure in each one (an app, a MySQL database, and a VPC):
```
└── magento
    ├── prod
    │   ├── app
    │   │   └── main.tf
    │   ├── mysql
    │   │   └── main.tf
    │   └── vpc
    │       └── main.tf
    ├── project-3
    │   ├── app
    │   │   └── main.tf
    │   ├── mysql
    │   │   └── main.tf
    │   └── vpc
    │       └── main.tf
    └── stage
        ├── app
        │   └── main.tf
        ├── mysql
        │   └── main.tf
        └── vpc
            └── main.tf
```    
The contents of each environment will be more or less identical, except perhaps for a few settings (eg. the prod environment may use bigger or more servers). As the size of the infrastructure grows, having to maintain all of this duplicated code between environments becomes more error prone. You can reduce the amount of copying and pasting using Terraform modules, but even the code to instantiate a module and set up input variables, output variables, providers and remote state can still create a lot of maintenance overhead.

Terragrunt allows you to keep your Magento backend configuration DRY (“Don’t Repeat Yourself”) by defining it once in a root location and inheriting that configuration in all child modules. Let’s say your Terraform code has the following folder layout:
```
stage
├── frontend-app
│   └── main.tf
└── mysql
    └── main.tf
``` 
To use Terragrunt, add a single terragrunt.hcl file to the root of your repo, in the stage folder, and one terragrunt.hcl file in each module folder:
```
stage
├── terragrunt.hcl
├── frontend-app
│   ├── main.tf
│   └── terragrunt.hcl
└── mysql
    ├── main.tf
    └── terragrunt.hcl
```
Now you can define your backend configuration just once in the root terragrunt.hcl file!


# Multi cloud deployments 

Terraform provides Magento 2 Open Source Cloud infrastructure as a code approach to provision and manage any cloud (AWS, GoogleCloud, Azure, Alibaba, or other types of services such as Kubernetes).

Terraform can manage popular service providers, such as AWS, GCP, Micosoft Azure, Alibaba Cloud, and VMware, as well as custom in-house and on-premises solutions.

## Enterprise Support/Installation/Development Package available.
Several Magento development Agencies select this custom cloud solution for their clients and they are willing to provide services/support for businesses based on this Open Source project.
This project currently has 10+ partners. 
If you are willing to be listed as cloud service provider feel free message me.

I also have Ansible Magento Cloud provisioning implementation:
https://github.com/nholuongut/magento-2-ansible

And also Magento Cloud provisioning Using AWS CDK. Coming soon ...


# Approximate Magento 2 AWS Cloud infrastructure Cost

```
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| Category    | Type                | Region    | Total cost | Count | Unit price | Instance type | Instance size |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| appservices | Email Service - 10K | us-west-2 | $1.00      | 1     | $1.00      |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| storage     | EFS storage – 20GB  | us-west-2 | $6.00      | 1     | $6.00      |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| storage     | S3 – 50Gb           | us-west-2 | $2.00      | 1     | $2.00      |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| compute     | ec2-Web Node        | us-west-2 | $61.20     | 1     | $61.20     | c5            | large         |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| networking  | elb - Load Balancer | us-west-2 | $43.92     | 2     | $21.96     |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| compute     | ec2-Admin-Cron Node | us-west-2 | $29.95     | 1     | $29.95     | t3            | medium        |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| database    | ElastiCache-Redis   | us-west-2 | $24.48     | 1     | $24.48     | t3            | small         |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| compute     | ec2-Varnish         | us-west-2 | $29.95     | 1     | $29.95     | t3            | large         |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| analytics   | ElasticSearch       | us-west-2 | $12.96     | 1     | $12.96     | t2            | micro         |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| database    | RDS MySQL           | us-west-2 | $48.96     | 1     | $48.96     | t3            | medium        |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
| storage     | EBS Storage 30Gb    | us-west-2 | $9.13      | 1     | $9.13      |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
|             |                     | Total     | $269.55    |       |            |               |               |
+-------------+---------------------+-----------+------------+-------+------------+---------------+---------------+
```
# eCommerce Cloud Price Visualisation 

![Magento 2 AWS Cloud Cost](small-big.png)

# Why not Magento Cloud?
```
+-----------------------------------------+-------------------------------------------+
|              Magento Cloud              |               This Solution               |
+-----------------------------------------+-------------------------------------------+
| Manual scaling, requires prior notice,  | Unlimited Resource, scaling by rule,      |
| vertical scaling,                       | no performance degradation                |
| performance degradation during scaling  |                                           |
+-----------------------------------------+-------------------------------------------+
| Fastly CDN only                         | Completely CDN agnostic,                  |
|                                         |  works with Cloudflare, CloudFront        |
+-----------------------------------------+-------------------------------------------+
| Works only with Enterprise version M2   | Works with any version of Magento 1/2     |
+-----------------------------------------+-------------------------------------------+
| Expensive $2000-$10000 month * +        | Paying only for AWS resources you used,   |
| Enterprise license                      | starting from 300$ months                 |
+-----------------------------------------+-------------------------------------------+
| Not Customizable                        | Fully Customizeble                        |
+-----------------------------------------+-------------------------------------------+
| Host only single Magento 2 CE           | Can host multiple project, web sites,     |
| installation                            | tech stacks, PHP, Node.JS, Python, Java;  |
|                                         | Magento 1/2, WordPres, Drupal, Joomla,    |
|                                         | Presta Shop, Open Cart, Laravel, Django   |
+-----------------------------------------+-------------------------------------------+
```
*Magento Cloud introduces: 
OVERAGE FEES for the Compute Overage usage (per vCPU day): ~$X(price of the Commerce Cloud is Adore Secret)/vCPU-day when a raw AWS vCPU cost is less than 1$ per day. 

From the Magento Cloud Agremment: 

Magento Cloud Customer hereby authorizes Magento, if applicable, to charge its credit card or other payment instrument or Subscription Fees, Overage Fees and/or any upgrades to the Services ordered, and any applicable taxes in arrears or at time of order, as the case may be.

Because of the bad Magento Cloud Architecture and performace you cloud HIDDEN OVERAGE FEES can be more then a Contract price. 


# Basic Deployment With CodeDeploy Example 

## Code and application deployment is beyond the scope of this repo. This repo for infrastructure provisioning only!!!

AWS CodeDeploy is a managed deployment technology. It provides great features like rolling deployments, automatic rollback, and load balancer integration. It is technology agnostic and Amazon uses it to deploy everything. 

ASSUMING YOU ALREADY HAVE an AWS account and CodeDeploy setup

Here are the basic that we take on a deployment for M2 

Here is the appspec.yml file (https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html#appspec-reference-ecs)
```
version: 0.0
os: linux
hooks:
    BeforeInstall:
        - location: config_files/scripts/beforeInstall.bash
          runas: root
    AfterInstall:
        - location: config_files/scripts/afterInstall.bash
          runas: mage_user
        - location: config_files/scripts/moveToProduction.bash
          runas: root
        - location: config_files/scripts/cacheclean.bash
          runas: mage_user
```

# Magento 2 AWS Code Deploy script example
Script to 'compile' magento on Deploy server - You pull and compile code to deploy server or build Docker container end after just push code to production using Code Deploy - fastest way 

```
cd production/build/public_html
git checkout .
git pull origin master
rm -rf var/cache/* var/page_cache/* var/composer_home/* var/tmp/*
php composer.phar update --no-interaction --no-progress --optimize-autoloader
bin/magento setup:upgrade
bin/magento setup:static-content:deploy -t Magento/backend
bin/magento setup:static-content:deploy en_US es_ES -a frontend
bin/magento setup:di:compile
# Make code files and directories read-only
echo "Setting directory base permissions to 0750"
find . -type d -exec chmod 0750 {} \;
echo "Setting file base permissions to 0640"
find . -type f -exec chmod 0640 {} \;
chmod o-rwx app/etc/env.php && chmod u+x bin/magento

# Compress source at shared directory
if [ ! -d /build ]; then
    mkdir -p /build
fi
tar -czvf /build/build.tar.gz . --exclude='./pub/media' --exclude='./.htaccess' --exclude='./.git' --exclude='./var/cache' --exclude='./var/composer_home' --exclude='./var/log' --exclude='./var/page_cache' --exclude='./var/import' --exclude='./var/export' --exclude='./var/report' --exclude='./var/backups' --exclude='./var/tmp' --exclude='./var/resource_config.json' --exclude='./var/.sample-data-state.flag' --exclude='./app/etc/config.php' --exclude='./app/etc/env.php'
```
Now you can deploy to your pre-configured group

```
sh ./compile.sh
aws deploy create-deployment \
--application-name AppMagento2 \
--deployment-config-name CodeDeployDefault.OneAtATime \
--deployment-group-name MyMagentoApp \
--description "Live Deployment" \
--s3-location bucket=mage-codedeploy,bundleType=zip,eTag=<tagname>,key=live-build2.zip
```

Create this script to show where you are in the deployment

show-deployment.sh

```
aws deploy get-deployment --deployment-id $1 --query "deploymentInfo.[status, creator]" --output text
```

File 'config_files/scripts/afterInstall.bash' should run setup:upgrade --keep-generated, nginx, php-fpm restart and similar stuff

##How to Deploy With Docker 

Just run command in your codeDeploy script 

```
docker pull [OPTIONS] MAGENTO_IMAGE_NAME[:TAG|@DIGEST]

```
Example of the deploy file: https://github.com/nholuongut/magento-2-cloud-on-aws-with-terraform/blob/master/deploy.sh

# Automate the installation of software using Golden AMI

A “golden AMI” or “gold image” is an Magento AMI you standardize through configuration, consistent security patching, and hardening. It also contains agents you approve for logging, security, performance monitoring, etc. Many enterprise customers have a mature AMI pipeline setup to create a golden AMI of base operating systems for the organization. For a sample golden AMI pipeline, see [The Golden AMI Pipeline] (https://aws.amazon.com/blogs/awsmarketplace/announcing-the-golden-ami-pipeline/).

You can launch an instance from an existing AMI, customize the instance, setup Software (Magento, ODDO, Wordpress, Shopware etc.) and then save this updated configuration as a custom AMI. Instances launched from this new custom AMI include the customizations that you made when you created the AMI.

# Magento 2 Installation Automation (Centos 8.2, AWS linux with ARM support) GitHub reposetory:

[Magento installation Script] (https://github.com/nholuongut/magento-linux-installation).

# Building an Golden AMI with Packer

Packer is an open-source tool by Hashicorp that automates the creation of machine images for different platforms. Developers specify the machine configuration using a JSON file called template, and then run Packer to build the image.


One key feature of Packer is its capability to create images targeted to different platforms, all from the same specification. This is a nice feature that allows you to create machine images of different types without repetitive coding.

You can get Packer and its documentation at the [Packer official site](https://www.packer.io/).  


# Use DynamoDb with Magento 2

Magento out of the box has a PHP Library to work with Dynamo DB. 

```
use Aws\DynamoDb\Exception\DynamoDbException;
use Aws\DynamoDb\Marshaler;

$sdk = new Aws\Sdk([
    'endpoint'   => 'http://localhost:8000',
    'region'   => 'us-west-2',
    'version'  => 'latest'
]);

$dynamodb = $sdk->createDynamoDb();
$marshaler = new Marshaler();

$tableName = 'Movies';

$year = 2015;
$title = 'The Big New Movie';

$item = $marshaler->marshalJson('
    {
        "year": ' . $year . ',
        "title": "' . $title . '",
        "info": {
            "plot": "Nothing happens at all.",
            "rating": 0
        }
    }
');

$params = [
    'TableName' => 'Movies',
    'Item' => $item
];

try {
    $result = $dynamodb->putItem($params);
    echo "Added item: $year - $title\n";

} catch (DynamoDbException $e) {
    echo "Unable to add item:\n";
    echo $e->getMessage() . "\n";
}

?>
```

You can record logs to a DynamoDB table with the AWS SDK and Monolog using /Monolog/Handler/DynamoDbHandler.php

When Time to Live (TTL) is enabled on a table in Amazon DynamoDB, a background job checks the TTL attribute of items to determine whether they are expired.

Also you can use the Amazon Web Services CloudWatch Logs Handler for Monolog library to integrate Magento 2 Monolog with CloudWatch Logs (https://github.com/nholuongut/cwh).

```
php composer.phar require nholuongut/cwh:^1.0
```

Terraform AWS moules maintained on my Github Repositories (https://github.com/nholuongut/terraform-for-aws)

All content, including [AWS Lab with Terraform](nholuongut/aws-lab-with-terraform) used in these configurations, is released under the MIT License. 

# Good news for the Magento Terraform Community 

Terragrunt issue with use modules from Terraform Registry is resolved now we can use many other modules! 
https://github.com/nholuongut/nholuong-terragrunt/issues

Terragrunt 31.5 release: Added support for fetching modules from any Terraform Registry using the new tfr:// protocol syntax for the source attribute. See the updated docs on source for more details.

# 🚀 I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)

![](https://i.imgur.com/waxVImv.png)
![](Donate.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License
* Nho Luong (c). All Rights Reserved.🌟
