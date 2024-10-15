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

## Exam questions
This section contains arguments likeky to be present in the exam.

### COPY vs ADD instructions
`COPY` lets you copy a **local** file or directory from your host to the Docker image.

`ADD` does the same thing, but if the source is a local tar archive, it is decompressed and extracted to the specified destination.

`ADD` also supports URLs as source, but this is discouraged, you should use *curl* or *wget* to fetch files from remote URLs.
