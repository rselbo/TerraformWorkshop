# Cloud Run and secrets 

Now we should have a Redis memorystore up and running so now we can create our cloud run deployment and make sure secrets are stored and passed to cloud run.

First we should make sure the secret from redis is stored in our secret manager. We dont need to create the secret manager, it is autmatically created once we enable the api with **google_project_service**, so go ahead and add **secretmanager.googleapis.com** to the list of services.

In googles secret manager we first need to create the secret to hold the secret value then we create a version that contains the actual secret value.
Terraform docs are your friend here [google](https://registry.terraform.io/providers/hashicorp/google/latest/docs) but what you need is **google_secret_manager_secret** and **google_secret_manager_secret_version**
The values we want to store are the **host**, **port** and **auth secret** for the redis instance. You can do this by creating a **google_secret_manager_secret** for each of them or by using a **for_each**

For the secret replication we can set it to automatic, and we need a depends_on as well 
```
  replication {
    automatic = true
  }
  depends_on = [google_project_service.services]
```

When we have configured the secrets we can apply them
```
terraform plan -var-file dev.tfvars -out dev.plan
terraform apply dev.plan
```

Once we have the secrets handled we can create the Cloud Run deployment. 
First we create a service account that we will use to run the cloud run deployment and set up the required IAM permissions for the runner. We also create an IAM member for allUsers so everyone can access the cloud run service.
This is very GCP specific, but all big cloud proviers have similar resources to provide/restrict access.
```
resource "google_service_account" "cloud_run_user" {
  project  = google_project.workshop_project.project_id
  account_id   = "workshop-service-account"
  display_name = "Workshop Service Account"
}

resource "google_project_iam_member" "cloud_run_user_iam" {
  project = google_project.workshop_project.project_id
  member  = "serviceAccount:${google_service_account.cloud_run_user.email}"
  role    = each.value

  for_each = toset([
    "roles/run.admin",
    "roles/iam.serviceAccountUser",
    "roles/secretmanager.secretAccessor",
    "roles/redis.editor",
  ])  
}

resource "google_cloud_run_service_iam_member" "public_member" {
  location = google_cloud_run_service.workshop_run.location
  project = google_cloud_run_service.workshop_run.project
  service = google_cloud_run_service.workshop_run.name
  role = "roles/run.invoker"
  member = "allUsers"
}
```

The final step is the cloud run deployment where we tie everything together. First we need to enable the api **run.googleapis.com** by adding it to the list of services we defined earlier.
Then we can create a **google_cloud_run_service** resource with **provider = google-beta**
This is where we will tie everything together, you can use the Cloud Run Service Basic example from the documentation as a starting point but for the image use **europe-north1-docker.pkg.dev/roars-workshop-root/roars-terraform-workshop/server**
Then we need to inject the secrets into the cloud run container, in this instance we will pass it through environment variables. So in the **google_cloud_run_service** inside the **containers** object we will add the following environment variables **REDIS_HOST**, **REDIS_PORT** and **REDIS_AUTH** and populate them with our secrets.
This is how you can add enviroment variables. (this is a part of the **google_cloud_run_service**)
```
      containers {
        image = "europe-north1-docker.pkg.dev/roars-workshop-root/roars-terraform-workshop/server"
        env {
          name = "REDIS_HOST"
          value_from {
            secret_key_ref {
              name = <your reference to the host secret>
              key = "latest"
            }
          }
        }
        env {
          name = "REDIS_PORT"
          ...
```

We also need to add the vpc connector, add the service account and a dependency on the service

```
  template {
    spec {
      service_account_name = google_service_account.cloud_run_user.email
      ...
    }

    metadata {
      annotations = {
        "autoscaling.knative.dev/maxScale" = "100"
        "run.googleapis.com/vpc-access-connector" = google_vpc_access_connector.workshop_vpc_connector.id
      }
    }
  }

  depends_on = [google_project_service.services]
```

And we can add an output value so we can see our cloud run URL without going to the cloud console.
```
output "workshop_run_addr" {
    value = google_cloud_run_service.workshop_run.status[0].url
}
```

Now you can plan and apply this change, then we are almost done.

[next](https://github.com/rselbo/TerraformWorkshop/blob/main/05-FinalSteps.md)


[previous](https://github.com/rselbo/TerraformWorkshop/blob/main/03-FirstInfrastructure.md)