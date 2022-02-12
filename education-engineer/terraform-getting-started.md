# Getting Started with Terraform - Docker Provider
HashiCorp Terraform is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share. You can then use a consistent workflow to provision and manage all of your infrastructure throughout its lifecycle.

The Docker provider is used to interact with Docker containers and images. It uses the Docker API to manage the lifecycle of Docker containers. The objective of this tutorial is to get you started with Terraform and walk you through the Terraform basics to build and destroy Docker infrastructure using Terraform Docker Provider. 

This tutorial provides you a step-by-step guide to achieve the following:

* Install Terraform on your local machine
* Validate Terraform installation
* Create basic Terraform configuration code using Docker provider
* Pull and create a docker image resource
* Create a docker container resource
* Clean up created resources 

## Prerequisites

* [Terraform](https://www.terraform.io/) **version** 0.13 or later
* [Docker](https://docs.docker.com/get-docker/)

### Install Terraform

HashiCorp distributes Terraform as a [binary package](https://www.terraform.io/downloads). Select the appropriate package for your system and download it as a zip archive.

After downloading Terraform, unzip the package. Terraform runs as a single binary named `terraform`. Any other files in the package can be safely removed and Terraform will still function.

Finally, make sure that the `terraform` binary is available on your `PATH`. This process will differ depending on your operating system.

#### Verify the installation

Verify that the installation worked by opening a new terminal session and listing Terraform's available subcommands.

```
$ terraform -help
Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands. If you're just getting started with Terraform, stick with the common commands. For other commands, please read the help and docs before usage.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure
  
##...
```

Add any subcommand to `terraform -help` to learn more about what it does and available options.

```
$ terraform -help plan
Usage: terraform [global options] plan [options]

  Generates a speculative execution plan, showing what actions Terraform
  would take to apply the current configuration. This command will not
  actually perform the planned actions.

  You can optionally save the plan to a file, which you can then pass to
  the "apply" command to perform exactly the actions described in the plan.

##...
```

#### Troubleshoot 
If you get an error that `terraform` could not be found, your PATH environment variable was not set up properly. Please go back and ensure that `your` PATH variable contains the directory where Terraform was installed.

**Linux or MacOs**
```shell
$ echo $PATH
```
Move the Terraform binary to one of the listed locations. This command assumes that the binary is currently in your downloads folder and that your `PATH` includes `/usr/local/bin`, but you can customize it if your locations are different.
```shell
mv ~/Downloads/terraform /usr/local/bin/
```
For more detail about adding binaries to your path, see this [Stack Overflow article](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux-unix).

**Windows**
[This Stack Overflow article](https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows) contains instructions for setting the PATH on Windows through the user interface.

### Install Docker 
In this tutorial Terraform interacts with Docker API to create and destroy Docker resources. To succesfully complete the tutorial, Docker must be installed on your local machine. [This Docker documentation](https://docs.docker.com/engine/install/) contains instuctions on downloading and installing Docker on your local machine. This process will differ depending on your operating system.  

**NOTE:** After successfully installing and starting Docker, the `dockerd` daemon runs with its default configuration. For troubleshooting and installation verification follow this [Docker guide](https://docs.docker.com/config/daemon/).

## Quick start tutorial
Now that you've installed Terraform and Docker, you can provision an NGINX server in less than a minute using Docker on Mac, Windows, or Linux. 

### Create Terraform configuration code 
We recommend you create a new directory on your local machine, and create the Terraform configutation file inside of the new directory.  

#### Create a directory named `terraform-demo`
```shell
mkdir terraform-demo
```
#### Change into the directory.
```shell
cd terraform-demo
```
#### Create a file to define your infrastructure.
```shell
touch main.tf
```
#### Open `main.tf `in your text editor, paste in the configuration below, and save the file.
**Linux or MacOs**
```hcl
# Set the required provider and versions
terraform {
  required_providers {
    # We recommend pinning to the specific version of the Docker Provider you're using
    # since new versions are released frequently
    docker = {
      source  = "kreuzwerker/docker"
      version = ">= 2.13.0"
    }
  }
}

# Configure the docker provider
provider "docker" {

}

# Pulls a docker image resource
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}

# Create a docker container resource
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 8000
  }
}
```
**Windows**
```hcl
# Set the required provider and versions
terraform {
  required_providers {
    # We recommend pinning to the specific version of the Docker Provider you're using
    # since new versions are released frequently
    docker = {
      source  = "kreuzwerker/docker"
      version = ">= 2.13.0"
    }
  }
}

# Configure the docker provider
provider "docker" { 
  host    = "npipe:////.//pipe//docker_engine"
  }

# Pulls a docker image resource
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}

# Create a docker container resource
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 8000
  }
}
```

### Initialize the directory
When you create a new configuration — or check out an existing configuration from version control — you need to initialize the directory with `terraform init`.

Initializing a configuration directory downloads and installs the providers defined in the configuration, which in this case is the docker provider.

```shell
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of kreuzwerker/docker from the dependency lock file
- Using previously-installed kreuzwerker/docker v2.16.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

### Create infrastructure
Apply the configuration now with the `terraform apply` command. Terraform will print output similar to what is shown below. We have truncated some of the output to save space.

```shell
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
##...

   + ports {
          + external = 8000
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

Before it applies any changes, Terraform prints out the execution plan which describes the actions Terraform will take in order to change your infrastructure to match the configuration.

The output format is similar to the diff format generated by tools such as Git. The output has a `+` next to `docker_container.nginx`, meaning that Terraform will create this resource. Beneath that, it shows the attributes that will be set. When the value displayed is (`known after apply`), it means that the value will not be known until the resource is created. For example, Docker assigns a random ID to images upon creation, so Terraform cannot know the value of the `id` attribute until you apply the change and the Docker provider returns that value.

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect or dangerous, it is safe to abort here with no changes made to your infrastructure.

In this case the plan is acceptable, so type `yes` at the confirmation prompt to proceed.

```shell
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 10s [id=sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5anginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=1043ae71d7a1cc61f2c2fcbc9ebe1975fc7a57f7f5dee71fd2159a5a57b44221]
```
You have now created infrastructure using Terraform! Visit `localhost:8000` in your web browser to verify the container started.

### Destroy infrastructure
You have now created and updated a Docker container on your machine with Terraform. In this section, you will use Terraform to destroy this infrastructure.

Once you no longer need infrastructure, you may want to destroy it to reduce your security exposure, costs, or resource overhead. For example, you may remove a production environment from service, or manage short-lived environments like build or testing systems. In addition to building and modifying infrastructure, Terraform can destroy or recreate the infrastructure it manages.

The terraform `destroy` command terminates resources managed by your Terraform project. This command is the inverse of terraform `apply` in that it terminates all the resources specified in your Terraform state. It does not destroy resources running elsewhere that are not managed by the current Terraform project.

```shell
$ terraform destroy

docker_image.nginx: Refreshing state... [id=sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5anginx:latest]
docker_container.nginx: Refreshing state... [id=1043ae71d7a1cc61f2c2fcbc9ebe1975fc7a57f7f5dee71fd2159a5a57b44221]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
##...

 # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5anginx:latest" -> null
      - latest      = "sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5a" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767" -> null
    }

  Plan: 0 to add, 0 to change, 2 to destroy.

  Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.
```

The `-` prefix indicates that the container will be destroyed. As with apply, Terraform shows its execution plan and waits for approval before making any changes.

Answer `yes` to execute this plan and destroy the infrastructure.

```shell
  Enter a value: yes

docker_container.nginx: Destroying... [id=1043ae71d7a1cc61f2c2fcbc9ebe1975fc7a57f7f5dee71fd2159a5a57b44221]
docker_container.nginx: Destruction complete after 1s
docker_image.nginx: Destroying... [id=sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5anginx:latest]
docker_image.nginx: Destruction complete after 0s
```


## Next Steps

Now that you have created your first infrastructure using Terraform, continue to the next [tutorial](https://learn.hashicorp.com/tutorials/terraform/docker-change) to modify your infrastructure.

Next, you will create real infrastructure in the cloud of your choice.

* [Amazon Web Services (AWS)](https://learn.hashicorp.com/tutorials/terraform/aws-build)
* [Azure](https://learn.hashicorp.com/tutorials/terraform/azure-build)
* [Google Cloud Platform (GCP)](https://learn.hashicorp.com/tutorials/terraform/google-cloud-platform-build)
* [Oracle Cloud Platform (OCI)](https://learn.hashicorp.com/tutorials/terraform/oci-build)

For more detail on the concepts used in this tutorial:

* Read about the Terraform configuration language in the [Terraform documentation](https://www.terraform.io/language).
* Learn more about Terraform [providers](https://www.terraform.io/language/providers).
* Find examples of other uses for Terraform in the documentation [use cases section](https://www.terraform.io/intro/use-cases).
* Read the [Docker provider documentation](https://registry.terraform.io/providers/kreuzwerker/docker/latest) to learn more.
