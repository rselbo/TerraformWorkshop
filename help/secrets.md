for_each secrets
```
resource "google_secret_manager_secret" "workshop_redis_secrets" {
  secret_id = format("workshop-redis-%s", each.value)
  for_each = toset([
    "host",
    "port",
    "auth"
  ])

  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_host" {
  secret = google_secret_manager_secret.workshop_redis_secrets["host"].id
  secret_data = google_redis_instance.workshop_redis.host
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_port" {
  secret = google_secret_manager_secret.workshop_redis_secrets["port"].id
  secret_data = google_redis_instance.workshop_redis.port
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_auth" {
  secret = google_secret_manager_secret.workshop_redis_secrets["auth"].id
  secret_data = google_redis_instance.workshop_redis.auth_string
}
```

individual secrets
```
resource "google_secret_manager_secret" "workshop_secret_redis_host" {
  secret_id = workshop-redis-host
  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret" "workshop_secret_redis_port" {
  secret_id = workshop-redis-port
  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret" "workshop_secret_redis_auth" {
  secret_id = workshop-redis-auth
  replication {
    automatic = true
  }
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_host" {
  secret = google_secret_manager_secret.workshop_secret_redis_host.id
  secret_data = google_redis_instance.workshop_redis.host
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_port" {
  secret = google_secret_manager_secret.workshop_secret_redis_port.id
  secret_data = google_redis_instance.workshop_redis.port
}

resource "google_secret_manager_secret_version" "workshop_secret_redis_auth" {
  secret = google_secret_manager_secret.workshop_secret_redis_auth.id
  secret_data = google_redis_instance.workshop_redis.auth_string
}
```