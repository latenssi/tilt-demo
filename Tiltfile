# -*- mode: Python -*

# Load an extension which allows creating kubernetes deployments without manifest files
# https://github.com/tilt-dev/tilt-extensions/tree/master/deployment
load("ext://deployment", "deployment_create")

# Create a kubernetes deployment for the demo app
# Tilt matches the image name to the one built later in this file
deployment_create(
  "demo",
  image="demo-image",
  ports="4000",
  env=[
    # deployment_create creates a kubernetes service, we can use its name here
    {"name":"DATABASE_HOST", "value":"postgres"}
  ],
  command=["bash", "-c", "mix ecto.setup && mix phx.server"]
)

# Create a kubernetes deployment for redis
# Pulls an image named "redis" from a public repo
deployment_create(
  "redis",
  ports="6379",
  readiness_probe={"exec":{"command":["redis-cli","ping"]}}
)

# Create a kubernetes deployment for postgres
# Pulls an image named "postgres" from a public repo
deployment_create(
  "postgres",
  ports="5432",
  readiness_probe={"exec":{"command":["pg_isready","--username=postgres","--dbname=demo_dev"]}},
  env=[
    {"name":"POSTGRES_DB", "value":"demo_dev"},
    {"name":"POSTGRES_USER", "value":"postgres"},
    {"name":"POSTGRES_PASSWORD", "value":"postgres"}
  ]
)

# Configures port-forwarding so that the demo app is reachable at localhost:4000
# Also adds resource_deps for postgres and redis so Tilt waits for those to be ready before running the demo app
k8s_resource("demo", port_forwards=4000, resource_deps=["postgres", "redis"])

# Build the demo app image, with in-place update of the container
docker_build("demo-image", "./demo", live_update=[
  sync("./demo", "/app"), # sync changed files to running container
  run("cd /app && mix deps.get", trigger=["./demo/mix.exs", "./demo/mix.lock"]),
  run("cd /app && mix ecto.setup", trigger=["./demo/priv/repo"])
])
