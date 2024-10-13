# Docker Notes
Notes for the [Docker Certified Associated](https://training.mirantis.com/certification/dca-certification-exam/) Exam.

### COPY vs ADD instructions
`COPY` lets you copy a **local** file or directory from your host to the Docker image.

`ADD` does the same thing, but if the source is a local tar archive, it is decompressed and extracted to the specified destination.

`ADD` also supports URLs as source, but this is discouraged, you should use *curl* or *wget* to fetch files from remote URLs.
