# 01 Getting Started

First we need to create the main files for starting a new project.
We need to create a **main.tf** and a **vars.tf**. The naming is not required since terraform reads all .tf files in the folder but this is the reccomended naiming. Variables are not required, but unless you only control a few resources you will find that vars help to avoid having to repeat values repeatedly.

**vars.tf** Create the file and create string variables for **project_id**, **project_region** and **environment**. Terraform allows any most symbols in the names, but the reccomended naming uses lowercase naming and _ for separating words.
This is how we define a variable in terraform, and we can multiple variable {} blocks
```
variable "variable_name" {
  type = string
}
```

**main.tf** Create the file and put in the providers. The providers below gives us access to all the google cloud features, all stable features are in the google provider but some of the new things are only avaliable in the google-beta provider.
You have to refer to the variable names you put into the **vars.tf** for the project and region fields below so everywhere you see a **var.** you have to put in the correct variable.
```
provider "google" {
  project     = var.
  region      = var.
}

provider "google-beta" {
  project     = var.
  region      = var.
}
```

Now we are ready to create the google project. The billing account information can be found on the console page, in a more structured setup that would not be necessary since it would be introduces some other way to avoid all projects having to refer to the billing accounts.
```
# -----------------------------------------------------------------------------
# Project
# -----------------------------------------------------------------------------
resource "google_project" "workshop_project" {
  name       = "Terraform workshop ${var.environment}"
  project_id = var.
  billing_account = "012345-6789AB-CDEF00" #https://console.cloud.google.com/billing shows the billing account in the form 012345-6789AB-CDEF00
}
```

The final step in our first steps is to create a storage bucket for where we want to store the terraform state after we have gotten this set up.
Here you have to create a globally uniqe name for the bucket. It has to be globally unique since if you configure access rights for it you can refer to the files with an url where the bucketname is part of the url.
I suggest naming it something like yourname-workshop-terraform-state-${var.environment}
```
resource "google_storage_bucket" "workshop_state" {
  name          = ""
  location      = "EU"
  force_destroy = true

  depends_on = [google_project.workshop_project]
}
```

To be able to easily swap between different projects/environments we create a **dev.tfvars** file and provide the values for the variables
The names on the left side must match your names from the **vars.tf** file. Also remember to rename the **yourname** pars of the project_id
```
project_id = "yourname-terraform-workshop-dev"
project_region = "europe-north1"
environment = "dev"
```

## Running terraform

Now we are finally ready to run some terraform commands. First we need to initialize terraform, that will download the providers and do some terraform setup in the background.
Once we have initialized the repo we create a new workspace and then we can run our first plan. Here we specify what variables to use with the -var-file and we save the plan to dev.plan with the -out parameter
```
terraform init
terraform workspace new dev
terraform plan -var-file dev.tfvars -out dev.plan
```

Then when we have a successful plan we should look over the plan that terraform printed. We should see that it will create a project and a bucket with some values that we specified and some that will be (known after apply). It is good practice to read over the plan and verify that it does what you expect. As an example; if you intended to just add a resource and you see that terraform wants to delete something you should verify that you have not accidentaly overwritten something. Sometimes changes requires a recreation of a resource but terraform will specify that in the plan as **resource will be replaced**.

When we are happy with the plan we can apply it, this will create the resources in our google project.
```
terraform apply dev.plan
```

