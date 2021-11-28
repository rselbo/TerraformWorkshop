```
resource "google_redis_instance" "workshop_redis" {
  name           = "workshop-redis"
  memory_size_gb = 1

  authorized_network = google_compute_network.workshop_redis_network.id
  connect_mode       = "PRIVATE_SERVICE_ACCESS"

  auth_enabled = true
  depends_on = [google_project_service.services]
}
```