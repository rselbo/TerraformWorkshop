# Setting up a shared state

We now hopefully have a google cloud project containing a storage bucket. But we have a small problem, the terraform state is on our local machine and this makes it difficult to cooporate with others. The best option is using Terraform Cloud, runatlantis or some other solution where the terraform planning/applying is tied to commits and the state is stored somewhere centrally so developers can't step on eachothers toes. We will do the second best option where we run terraform manually but we store the terraform state in a central bucket so it wont be lost if your computer dies. Again in a proper setup we would have the bucket in a separate project so we dont have to do this in 2 steps.

Terraform has a good system for this by specifying a backend, this is usually done in **backend.tf**
Unfortunately we can't use a variable in the backend so we cant specify **google_storage_bucket.workshop_state.name** to get the name so here we have to manually copy the bucket name from the bucket we created.
```
terraform {
  backend "gcs" {
    bucket  = ""
    prefix  = "/workshop"
  }
}
```

Since we have changed the backend we need to initialize again. If you try to run a plan or apply terraform will give you an error that tells you that you need to reinitialize terraform.
When you run the init it will detect that you have an existing terraform state and a new backend so it will ask if you want to copy existing state to the new backend? Answer yes to this question.
```
terraform init
```

Now anyone who has access to the project can run terraform init, plan and apply and change the infrastructure and as long as everybody has the same .tf files there will be no problems.

[next](https://github.com/rselbo/TerraformWorkshop/blob/main/03-FirstInfrastructure.md)


[previous](https://github.com/rselbo/TerraformWorkshop/blob/main/01-FirstSteps.md)
