# go-live-reload

- start with `Dockerfile`:
```shell
# Create another stage called "dev" that is based off of our "base" stage (so we have golang available to us)
FROM base as dev

# Install the air binary so we get live code-reloading when we save files
RUN curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s -- -b $(go env GOPATH)/bin

# Run the air command in the directory where our code will live
WORKDIR /opt/app/api
CMD ["air"]
```

Create a new file called `docker-compose.yml`:

```shell
version: "3"
services:
  app:
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - .:/opt/app/api
```

- run the command to check if everything is healthy
```shell
docker-compose build
```

- You now should have a new container built that has Go and Air installed into it. So let's create a simple Go program and see if we can get it to recompile on save with Air.
Initial Code
- To get the party started, let's add a simple main.go to our root directory of the new project.
```shell
package main

import (
  "fmt"
  "time"
)

func main() {
  for {
    fmt.Println("Hello World")
    time.Sleep(time.Second * 3)
  }
}
```

- This simple program prints "Hello World" every 3 seconds. The reason we're doing this is because we want to see a "long lived" start in our container so we can live reloading work later with Air.
- Let's also init our Go module from within the container, run the follow command (replace USER with your GitHub user please):
```shell 
docker-compose run --rm app go mod init github.com/MaxwelMazur/go-live-reload
```


- We need to create a .air.toml file that Air will read as well. Air provides a simple command for us to use, which we can run from inside of our new shiny container built from Docker Compose.
Air Setup
- We should run our Air init from inside of our app container (defined in our compose yaml file). To do this, let's run the follow command:

```shell
docker compose run --rm app air init
```


- Note: I prefer using run with the --rm flag because I don't like having a bunch of random containers laying around from commands I've run in them.

- You should see output similar to:

```shell
docker-compose run --rm app air init
```


Let's start up our container and see our project come alive.
```shell
$ docker compose up
```
