# First infrastructure

Now that we have the basic framework up and running we can start setting up the real infrastructure of our project.
The project consists of a Cloud Run deployment that talks to a Redis memorystore and we pass secrets from redis to the Cloud Run using googles Secret Manager. This is where one of the real strengths of terraform becomes apparent, we can automate the setup of the resources **and** automatically share secrets and variables of the resources we create. This makes it easy to make sure that we always have strong passwords and secrets within the projects since they can be autmatically created and sent to the resources that need it.

Since we have a new project we need to make sure we enable the apis we need for our projects, in google we do this with **google_project_service**.
Here we use a new feature **for_each**, terraform will "unroll" this loop and automatically create a new resource for each api that is enabled here. Right now we enable the redis and 2 unrelated services that is needed for some GCP specific networking setup. We also refer to the project using the **google_project.workshop_project.project_id** this is mostly to show that we can refer to other resources we create, we could also have used **var.project_id** here.
```
resource "google_project_service" "services" {
  project = google_project.workshop_project.project_id
  service = each.value
  for_each = toset([
    "redis.googleapis.com",
    "servicenetworking.googleapis.com",
    "vpcaccess.googleapis.com",
  ])
}
```

Then we can set up our redis instance. In this case since we want to let our serverless cloud run access redis we need to create some GCP specific networking.
We create a compute network, and some resources that allows us to connect the different server/serverless services that otherwise could not talk to eachother.
You should also note the **depends_on** here, terraform builds an implicit dependency chain when you refer to other resources, but in this case we need to add an explisit dependency so we don't run into problems where terraform tries to create the compute_network before the services are enabled. Since we don't refer directly to the services terraform can not infer this dependency.
```
resource "google_compute_network" "workshop_redis_network" {
  name = "workshop-redis-network"
  auto_create_subnetworks = true
  depends_on = [google_project_service.services]
}

resource "google_compute_global_address" "service_range" {
  name          = "address"
  purpose       = "VPC_PEERING"
  address_type  = "INTERNAL"
  prefix_length = 16
  network       = google_compute_network.workshop_redis_network.id
}

resource "google_service_networking_connection" "private_service_connection" {
  network                 = google_compute_network.workshop_redis_network.id
  service                 = "servicenetworking.googleapis.com"
  reserved_peering_ranges = [google_compute_global_address.service_range.name]
}

resource "google_vpc_access_connector" "workshop_vpc_connector" {
  name = "workshop-vpc-connector"
  ip_cidr_range = "10.8.0.0/28"
  network = google_compute_network.workshop_redis_network.name
}
```

For the actual redis instance you need to find the resource definition for redis in the terraform docs for [google](https://registry.terraform.io/providers/hashicorp/google/latest/docs), the basic example setup for redis is a good enough for us but we need a couple of extra fields due to our networking and security needs so add the fields below to the redis resource.
```
  authorized_network = google_compute_network.workshop_redis_network.id
  connect_mode       = "PRIVATE_SERVICE_ACCESS"
  auth_enabled = true
  depends_on = [google_project_service.services, google_compute_network.workshop_redis_network]
```

Once you have everything ready we can plan and apply our changes.
```
terraform plan -var-file dev.tfvars -out dev.plan
terraform apply dev.plan
```

[next](https://github.com/rselbo/TerraformWorkshop/blob/main/04-CloudRunAndSecrets.md)


[previous](https://github.com/rselbo/TerraformWorkshop/blob/main/02-PrepareBackend.md)
