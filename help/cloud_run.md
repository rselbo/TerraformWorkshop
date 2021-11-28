```
resource "google_cloud_run_service" "workshop_run" {
  provider = google-beta
  name     = "workshop-run"
  location = var.project_region

  template {
    spec {
      service_account_name = google_service_account.cloud_run_user.email
      containers {
        image = "europe-north1-docker.pkg.dev/roars-workshop-root/roars-terraform-workshop/server"
        env {
          name = "REDIS_HOST"
          value_from {
            secret_key_ref {
              name = google_secret_manager_secret.workshop_redis_secrets["host"].secret_id
              key = "latest"
            }
          }
        }
        env {
          name = "REDIS_PORT"
          value_from {
            secret_key_ref {
              name = google_secret_manager_secret.workshop_redis_secrets["port"].secret_id
              key = "latest"
            }
          }
        }
        env {
          name = "REDIS_AUTH"
          value_from {
            secret_key_ref {
              name = google_secret_manager_secret.workshop_redis_secrets["auth"].secret_id
              key = "latest"
            }
          }
        }
      }
    }
    metadata {
      annotations = {
        "run.googleapis.com/vpc-access-connector" = google_vpc_access_connector.workshop_vpc_connector.self_link
      }
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }

  depends_on = [google_project_iam_member.cloud_run_user_iam]
}
```
