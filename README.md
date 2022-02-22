# Forked

This project has been forked to allow the driver to NOT append
bucket=bucketname when the bucket= prarameter has already been given in
DEFAULT_S3FSOPTS

We noticed that having a `name: "bucket"` variable in the docker compose yml file created a volume with the name "bucket", which would then not be removed when the
compose project was brought down

Not having this fixed name, allows us to lock the volume to a bucket - behavior which you ideally would want, because you want to limit the buckets a volume can write to

## Usage is as follows

For each bucket we want to access we create a dedicated plugin instance
In our use case we usually only want to access a single bucket

### Create the plugin and bind the url/bucket

```shell
docker plugin install --alias s3fs-my-bucket \
  screencom/s3fs-volume-plugin \
  --grant-all-permissions --disable

docker plugin set s3fs-my-bucket AWSACCESSKEYID=YOURKEYHERE
docker plugin set s3fs-my-bucket AWSSECRETACCESSKEY=SUPERSECRET
docker plugin set s3fs-my-bucket DEFAULT_S3FSOPTS=nomultipart,use_path_request_style,url=https://minio.location.tld,bucket=my-bucket
docker plugin enable s3fs-my-bucket
```

### Create a volume on the command line to use somewhere

```shell
docker volume create -d s3fs-my-bucket s3fs-my-bucket-volume
```

Then either use the volume in docker run or in a compose yml

We noticed that using the volume in a docker compose file while using
docker-compose gave an error after (re)starting the stack (docker-compose up
when the stacks is running) and also didn't remove the volume on bringing the stack
down

When using the new docker compose plugin, this behavior was fixed

```yml
version: "3"
services:
  container:
    image: ubuntu
    volumes:
      - client:/cdn/client
volumes:
  client:
    driver: s3fs-my-bucket
```

Using `docker compose up -d` created the volume, `docker compose down` removed it again

Exactly as we expect it to.

And now, back to the original README

# Docker Managed Volume Plugins

This project provides managed volume plugins for Docker to connect to [CIFS](https://github.com/marcelo-ochoa/docker-volume-plugins/tree/master/cifs-volume-plugin), [GlusterFS](https://github.com/marcelo-ochoa/docker-volume-plugins/tree/master/glusterfs-volume-plugin) [NFS](https://github.com/marcelo-ochoa/docker-volume-plugins/tree/master/nfs-volume-plugin).

Along with a generic [CentOS Mounted Volume Plugin](https://github.com/marcelo-ochoa/docker-volume-plugins/tree/master/centos-mounted-volume-plugin) that allows for arbitrary packages to be installed and used by mount.

There are two key labels

- `dev` this is an unstable version primarily used for development testing, do not use it on production.
- `latest` this is the latest version that was built which should be ready for use in production systems.

**There is no robust error handling. So garbage in -> garbage out**
