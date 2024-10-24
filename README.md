# Docker Notes
Notes for the [Docker Certified Associated](https://training.mirantis.com/certification/dca-certification-exam/) Exam.

## General info
This section contains non-trivial informations I learned while studying for the exam.

### Healthcheck
Container healthcheck monitoring can be defined in the Dockerfile with the `HEALTHCHECK` instruction as follows:
```dockerfile
HEALTHCHECK [OPTIONS] CMD <command>
```
The following options are available:

| Option           | Default value | Notes                                           |
| ---------------- | ------------- | ----------------------------------------------- |
| --interval       | 30s           |                                                 |
| --timeout        | 30s           |                                                 |
| --start-period   | 0s            | Ignore healthckeck failure while starting       |
| --start-interval | 5s            | Available on Dokcer Engine >= 25.0              |
| --retries        | 3             | Consecutive failures to be considered unhealthy |

The command's exit status indicates the health status of the container. The possible values are:
- **0**: success - the container is healthy
- **1**: unhealthy - the container is not healthy
- **2**: reserved - don't use this exit code

Example:
```dockerfile
HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
```

The following flags for the `docker run` command can override the healthcheck defined in the Docker image:

| Option                  | Description                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------- |
| --health-cmd            | Command to run to check health                                                        |
| --health-interval       | Time between running the check                                                        |
| --health-timeout        | Maximum time to allow one check to run                                                |
| --health-start-period   | Start period for the container to initialize before starting health-retries countdown |
| --health-start-interval | Time between running the check during the start period                                |
| --health-retries        | Consecutive failures needed to report unhealthy                                       |
| --no-healthcheck        | Disable any container-specified HEALTHCHECK                                           |

### Docker commit
If you make changes inside a container you can commit the container's file changes or settings into a new image:
```
docker container commit <container-id> <image-tag>
```

Use the `--change` option (or `-c`) to apply Dockerfile instructions to the created image, for example:
```
docker commit -c='CMD ["sh"]' -c "EXPOSE 80" -c "ENV DEBUG=true" c3f279d17e0a  myimage:v2
```

By default the container is paused during commit.

Use `--pause false` option to avoid pausing the container during commit.

### Image layers
Use the following command to view all the layers of a Docker image:
```
dokcer image history <image-id>
```

Every container adds a writable layer on top of its image.

### Inspecting Docker images
Use `docker image inspect` to inspect images. use the `--format` option (or `-f`) to print only specific informations.

Print image id:
```
dokcer image inspect -f {{.Id}}
```

Print container config as JSON:
```
dokcer image inspect -f {{json .ContainerConfig}}
```

Print container hostname:
```
dokcer image inspect -f {{.ContainerConfig.Hostname}}
```

### Pruning Docker images
Command `docker image prune` removes all dangling images (not tagged and not referenced by any container).

Use the `--all` option (or `-a`) to remove all images not referenced by any container.

### Flattening Docker images
Images can be flattened to a single layer. This might be usefull to reduce an image overall size. The trick is to export the image from a container and import it back:
```
docker run -d --name mycontainer mysql
docker container export --output flattened-image.tar mycontainer
cat flattened-image.tar | docker image import - myimage:latest
```

### Export Docker images
Use `docker image save` to export an image with all tags and all layers.
```
docker image save ubuntu > ubuntu.tar
```
You can select only specific tags:
```
 docker save -o ubuntu.tar ubuntu:lucid ubuntu:saucy
```
Use `docker image load` to restore the image:
```
docker image load --input ubuntu.tar
```

### Docker search
You can search for images on Docker Hub using the `docker search` command.

Use the `--filter` option to add some filters and the `--limit` option to limit the result.
```
docker search --filter stars=10 --filter is-official=true --limit 10 debian
```

### Networking
We have three network drivers: **bridge** (default), **host** and **none**.

#### Bridge
Docker lets containers connected to the same bridge network communicate, while providing isolation from containers that aren't connected to that bridge network.

A default bridge network is created automatically, and newly-started containers connect to it unless otherwise specified.

**User defined bridge networks are superior** to the default bridge network:
- User-defined bridges provide automatic DNS resolution between containers
- Containers can be attached and detached from user-defined networks on the fly
- User-defined networks can be configured at creation time
- Linked containers on the default bridge network share environment variables

#### Host
The container's network stack isn't isolated from the Docker host and the container doesn't get its own IP-address. For instance, if you run a container which binds to port 80 and you use host networking, the container's application is available on port 80 on the host's IP address.

Usefull for network-monitoring applications running in a Docker container.

#### None
The container is completely isolated from other containers and the external world.

## Docker Swarm
This section contains info about Docker Swarm

### Manager-only nodes
The manager nodes in a swarm cluster can be assigned tasks. It is recommended to drain manager nodes so that they won't run any service and can use all the available resources for manager-related activities.
```
docker node update --availability drain <node-id>
```

### Forcing new cluster
In case manager quorum is lost and the lost manager nodes are unrecoverable the recommended way to restore the cluster is to re-initialize the cluster on a running manager node:
```
docker swarm init --force-new-cluster --advertise-addr <ip-addr>:2377
```
The state of the cluster will be recovered from the distributed state store.

## Exam questions
This section contains arguments likeky to be present in the exam.

### COPY vs ADD instructions
`COPY` lets you copy a **local** file or directory from your host to the Docker image.

`ADD` does the same thing, but if the source is a local tar archive, it is decompressed and extracted to the specified destination.

`ADD` also supports URLs as source, but this is discouraged, you should use *curl* or *wget* to fetch files from remote URLs.

### Multiple ways to scale Swarm services
There are two approaches for scaling a service to the desired number of replicas. The following commands are equivalent:
```
docker service scale service01=3
docker service update --replicas 3 service01
```
The difference is that with `docker service scale` you can scale multiple services at once:
```
docker service scale service01=3 service02=5
```

### Overlay network encryption
To enable encryption on overlay network use the `--opt encrypted` flag a when creating the network:
```
docker network create --opt encrypted --driver overlay mynetwork
```
Docker creates IPSEC tunnels between the nodes to communicate. The manager nodes automatically rotate encryption keys every 12 hours.

Encryption is not supported on Windows. Windows nodes won't be able to communicate, but no error will be detected.

### Creating containers and services using templates
When creating a service or a container you can use templates on some options. Example:
```
docker service create --hostname="{{{{.Node.Hostname}}-{{.Service.Name}}" ubuntu
```
These are all the valid placeholder values:
- .Service.ID
- .Service.Name
- .Service.Labels
- .Node.ID
- .Node.Hostname
- .Task.ID
- -Task.Name
- .Task.Slot

Placeholders are only supported for the following flags:
- --hostname
- --env
- --mount
