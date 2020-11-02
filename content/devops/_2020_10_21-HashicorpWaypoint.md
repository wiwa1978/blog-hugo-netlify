
https://medium.com/@michael_41345/getting-started-with-hashicorp-waypoint-how-to-create-a-python-app-using-flask-docker-80b279c88b2a


```
~ ❯ brew install hashicorp/tap/waypoint                                                                                                                                                 20s 08:57:04
==> Installing waypoint from hashicorp/tap
==> Downloading https://releases.hashicorp.com/waypoint/0.1.3/waypoint_0.1.3_darwin_amd64.zip
######################################################################## 100.0%
🍺  /usr/local/Cellar/waypoint/0.1.3: 3 files, 133.6MB, built in 10 seconds
```

```
Welcome to Waypoint
Docs: https://waypointproject.io
Version: v0.1.3

Usage: waypoint [-version] [-help] [-autocomplete-(un)install] <command> [args]

Common commands
  build        Build a new versioned artifact from source
  deploy       Deploy a pushed artifact
  release      Release a deployment
  up           Perform the build, deploy, and release steps for the app

Other commands
  artifact        Artifact and build management
  config          Application configuration management
  context         Server access configurations
  deployment      Deployment creation and management
  destroy         Delete all the resources created for an app
  docs            Show documentation for components
  exec            Execute a command in the context of a running application instance
  hostname        Application URLs
  init            Initialize and validate a project
  install         Install the Waypoint server to Kubernetes, Nomad, or Docker
  logs            Show log output from the current application deployment
  runner          Runner management
  server          Server management
  token           Authenticate and invite collaborators
  ui              Open the web UI
  version         Prints the version of this Waypoint CLI
```


```
~ ❯ docker pull hashicorp/waypoint:latest                                                                                                                                                   08:59:52
latest: Pulling from hashicorp/waypoint
df20fa9351a1: Pull complete
7004088794d4: Pull complete
10a5f87a56ed: Pull complete
Digest: sha256:9393a464966957834d08d2e6b7b9eaabc1e908b0862bd9d02ef44891cb53e505
Status: Downloaded newer image for hashicorp/waypoint:latest
docker.io/hashicorp/waypoint:latest
```

```
~ ❯ waypoint install --platform=docker -accept-tos                                                                                                                                      18s 09:00:40
 + Installing Waypoint server to docker
 + Server container started!
 + Configuring server...
Waypoint server successfully installed and configured!

The CLI has been configured to connect to the server automatically. This
connection information is saved in the CLI context named "install-1603263687".
Use the "waypoint context" CLI to manage CLI contexts.

The server has been configured to advertise the following address for
entrypoint communications. This must be a reachable address for all your
deployments. If this is incorrect, manually set it using the CLI command
"waypoint server config-set".

Advertise Address: waypoint-server:9701
Web UI Address: https://localhost:9702
```


```
~ ❯ waypoint token new                                                                                                                                                                      09:03:33
bM152PWkXxfoy4vA51JFhR7LsUvybSKCUu3HFM1GtWvTCRCVEEMmnMGR1E5ci6LiFzqtcdEX5UvGfpJHRCcQWsEjVsS4XH8pptdma
```

```
~/SynologyDrive/Programming/Waypoint ❯ git clone https://github.com/hashicorp/waypoint-examples.git                                                                                         09:02:07
Cloning into 'waypoint-examples'...
remote: Enumerating objects: 335, done.
remote: Counting objects: 100% (335/335), done.
remote: Compressing objects: 100% (214/214), done.
remote: Total 1061 (delta 145), reused 250 (delta 93), pack-reused 726
Receiving objects: 100% (1061/1061), 1.25 MiB | 3.13 MiB/s, done.
Resolving deltas: 100% (348/348), done.
```

```
~/SynologyDrive/Programming/Waypoint ❯ cd waypoint-examples/docker/nodejs                                                                                                                   09:05:21
~/SynologyDrive/Programming/Waypoint/waypoint-examples/docker/nodejs main ❯ waypoint init                                                                                                   09:05:31
 + Configuration file appears valid
 + Connection to Waypoint server was successful
 + Project "example-nodejs" and all apps are registered with the server.
 + Plugins loaded and configured successfully
 + Authentication requirements appear satisfied.

Project initialized!

You may now call 'waypoint up' to deploy your project or
commands such as 'waypoint build' to perform steps individually.
```

```
~/SynologyDrive/Programming/Waypoint/waypoint-examples/docker/nodejs main ❯ waypoint up                                                                                                     09:05:39

» Building...
Creating new buildpack-based image using builder: heroku/buildpacks:18
 + Creating pack client
 + Building image
 │ [exporter] Adding 1/1 app layer(s)
 │ [exporter] Adding layer 'launcher'
 │ [exporter] Adding layer 'config'
 │ [exporter] Adding label 'io.buildpacks.lifecycle.metadata'
 │ [exporter] Adding label 'io.buildpacks.build.metadata'
 │ [exporter] Adding label 'io.buildpacks.project.metadata'
 │ [exporter] *** Images (d754a44bdfb5):
 │ [exporter]       index.docker.io/library/example-nodejs:latest
 │ [exporter] Adding cache layer 'heroku/nodejs-engine:nodejs'
 │ [exporter] Adding cache layer 'heroku/nodejs-engine:toolbox'
 + Injecting entrypoint binary to image

Generated new Docker image: example-nodejs:latest

» Deploying...
 + Setting up waypoint network
 + Starting container
 + App deployed as container: example-nodejs-01EN4ZT65E5X8CFX5F8HFBZVWS

» Releasing...

The deploy was successful! A Waypoint deployment URL is shown below. This
can be used internally to check your deployment and is not meant for external
traffic. You can manage this hostname using "waypoint hostname."

           URL: https://supposedly-lucky-sole.waypoint.run
Deployment URL: https://supposedly-lucky-sole--v1.waypoint.run

```