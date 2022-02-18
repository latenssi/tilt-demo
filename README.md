# Kind + Tilt demo

A simple demo showcasing running a development environment for a Phoenix app in a local kubernetes cluster leveraging Kind and Tilt. The dev enviroment consist of:

- the Phoenix app
- a PostgreSQL server
- Redis

_Redis is not actually used in the app, it is only added for demonstrative purposes._

## Pre-req:

- Golang
- Docker

```bash
# Install kind
go install sigs.k8s.io/kind@v0.11.1

# Install ctlptl
go install github.com/tilt-dev/ctlptl/cmd/ctlptl@latest

# Create a cluster and a local image repository
ctlptl create cluster kind --registry=ctlptl-registry

# Run dev environment
tilt up
```

You should now have the complete dev enviroment set up.

If you now press `space` you should be directed to a webpage showing the Tilt dashboard.

**Test live reload:**

1. Navigate to http://localhost:4000
2. Change something in `./demo/lib/demo_web/templates/page/index.html.heex`, you should see the change immediately in the browser
3. Same should apply to any code in the `./demo` folder

**Disclaimer:**

Due to my lack of knowledge in Phoenix, the current _Tiltfile_ is not catching datamodel changes properly. If you change a model and the change requires a database migration, please make sure the `mix ecto.setup` is run. Currently the command is run once on startup and whenever there is a change in `./demo/priv/repo`.

Also the application might not reload properly when code changes.
