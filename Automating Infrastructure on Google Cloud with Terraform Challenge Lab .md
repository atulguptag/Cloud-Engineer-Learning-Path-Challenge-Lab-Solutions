## `Lab Name` - *Automating Infrastructure on Google Cloud with Terraform: Challenge Lab [GSP345]*
## `Lab Link` - [Click Here](https://www.cloudskillsboost.google/focuses/42740?parent=catalog)

## [YouTube Solution Link](https://youtu.be/wgAsPoGLbKE)
## Task 1. Create the configuration files
Run the below commands in the cloud shell terminal

```cmd
touch main.tf
touch variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf
touch outputs.tf
touch variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf
touch outputs.tf
touch variables.tf
cd
```

* (Paste it in variable.tf )

```cmd
variable "region" {
 default = "YOUR_REGION"
}

variable "zone" {
 default = "YOUR_ZONE"
}

variable "project_id" {
 default = "PROJECT_ID"
}
```

* (Paste it in main.tf )

```cmd
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}

module "instances" {
  source     = "./modules/instances"
}
```


Run this in CloudShell : 

```cmd
terraform init 
```
#############################################################################

## Task 2. Import infrastructure

* (Paste it in modules/instances/instances.tf )

```cmd
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         =  "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

Run this in CloudShell

```cmd
terraform import module.instances.google_compute_instance.tf-instance-1 [INSTANCE_ID_1]
```

```cmd
terraform import module.instances.google_compute_instance.tf-instance-2 [INSTANCE_ID_2]
```

```cmd
terraform plan
terraform apply
```

#############################################################################

## Task 3. Configure a remote backend

* Paste the below code in storage/storage.tf

```cmd
resource "google_storage_bucket" "storage-bucket" {
  name          = "YOUR_BUCKET_NAME"
  location      = "us"
  force_destroy = true
  uniform_bucket_level_access = true
}
```

* Add following to main.tf

```cmd
module "storage" {
  source     = "./modules/storage"
}
```

Run this in CloudShell

```cmd
terraform init
terraform apply
```

* Update following code to main.tf

```cmd
terraform {
  backend "gcs" {
    bucket  = "YOUR_BUCKET_NAME"
 prefix  = "terraform/state"
  }
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "4.53.0"
    }
  }
}
```

* Run this in CloudShell

```cmd
terraform init
```


#############################################################################

## Task 4. Modify and update infrastructure

* Add following to instance.tf

```cmd
resource "google_compute_instance" "INSTANCE_NAME" {
  name         = "YOUR_INSTANCE_NAME"
  machine_type = "n1-standard-2"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

* Run this in CloudShell

```cmd
terraform init
terraform apply
```

#############################################################################

## Task 5. Destroy resources

```cmd
terraform taint module.instances.google_compute_instance.INSTANCE_NAME
```

```
terraform init
terraform apply
```

* Go and `remove instance-3` from `instance.tf`

```cmd
terraform apply
```

#############################################################################

## Task 6. Use a module from the Registry


* (Paste it in main.tf)

```cmd
module "vpc" {
    source  = "terraform-google-modules/network/google"
    version = "~> 6.0.0"

    project_id   = "PROJECT_ID"
    network_name = "VPC_NAME"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-east1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-east1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "Subscribe TO ATUL GUPTA"
        },
    ]
}
```

* Run this in CloudShell

```cmd
terraform init
terraform apply
```

* Go to Instance.tf and Update ALL with the following

```cmd
resource "google_compute_instance" "tf-instance-1"{
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         ="ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "VPC_NAME"
     subnetwork = "subnet-01"
  }
  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}

resource "google_compute_instance" "tf-instance-2"{
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "ZONE"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
    network = "VPC_NAME"
     subnetwork = "subnet-02"
  }

  metadata_startup_script = <<-EOT
        #!/bin/bash
    EOT
  allow_stopping_for_update = true
}
```

* Run this in CloudShell

```cmd
terraform init
terraform apply
```

#############################################################################

## Task 7. Configure a firewall

* Add the following in main.tf

```cmd
resource "google_compute_firewall" "tf-firewall"{
  name    = "tf-firewall"
 network = "projects/YOUR_PROJECT_ID/global/networks/YOUR_VPC_Name"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}
```

* Run this in CloudShell

```cmd
terraform init
terraform apply
```

# Congratulations🎉! You're all done with this Challenge Lab.