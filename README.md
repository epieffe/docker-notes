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

## Exam questions
This section contains arguments likeky to be present in the exam.

### COPY vs ADD instructions
`COPY` lets you copy a **local** file or directory from your host to the Docker image.

`ADD` does the same thing, but if the source is a local tar archive, it is decompressed and extracted to the specified destination.

`ADD` also supports URLs as source, but this is discouraged, you should use *curl* or *wget* to fetch files from remote URLs.
