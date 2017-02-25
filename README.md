[![license](https://img.shields.io/github/license/monder/goofys-docker.svg?maxAge=2592000&style=flat-square)]()
[![GitHub tag](https://img.shields.io/github/tag/monder/goofys-docker.svg?style=flat-square)]()

goofys-docker is a docker [volume plugin] wrapper for S3

## Overview

The initial idea behind mounting s3 buckets as docker volumes is to provide store for configs and secrets. The volume as per [goofys] does not have features like random-write support, unix permissions, caching.

## Getting started

### Requirements

The docker host should have [FUSE] support with `fusermount` cli utility in `$PATH`

### Building

There are prebuilt binaries availble [here][download]. If you need to build it yourself there is a helper file `build.sh` that will run a container that builds the application using go 1.5. Version 1.5 is used to workaround https://github.com/docker/docker/issues/20865

### Configuration

Currently there is no support for configuration options, but the defaults are reasonable for most of the cases.
The most simple way to configure aws credentials is to use [IAM roles] to access the bucket for the machine, [aws configuration file][AWS auth] or [ENV variables][AWS auth]. The credentials will be used for all buckets mounted by `goofys-docker`.

### Running

```
./goofys-docker
```
The socket `/run/docker/plugins/goofys.sock` will be created to interact with docker. Ownership of the file is `root:wheel`

### Using with docker

```
docker volume create --name=VOLUME_NAME --driver=goofys --opt OPTION
```

#### Options

* `bucket` - Optional S3 bucket name. The default bucket is the volume name.
* `prefix` - Optional S3 prefix path.
* `region` - Optional AWS region (default is "us-east-1").
* `debugs3` - Optional S3 debug logs (default is 0).

Create a new volume by issuing a docker volume command:
```
docker volume create --name=test-docker-goofys --driver=goofys region=eu-west-1
```
That will create a volume connected to `test-docker-goofys` bucket. The region of the bucket will be autodetected.

Nothing is mounted yet.

Launch the container with `test-docker-goofys` volume mounted in `/home` inside the container.
```
docker run -it --rm -v test-docker-goofys:/home:ro -it busybox sh
/ # cat /home/test
test file content
/ # ^D
```

Pass the bucket name as an option instead of the default volume name value:
```
docker volume create --name=vol1 --driver=goofys --opt bucket=test-docker-goofys --opt region=eu-west-1
docker run -it --rm -v vol1:/home:ro -it busybox sh
/ # cat /home/test
test file content
/ # ^D
```

It is also possible to mount a subfolder:
```
docker volume create --name=vol1 --driver=goofys --opt prefix=folder region=eu-west-1
docker run -it --rm -v vol1:/home:ro -it busybox sh
/ # cat /home/test
test file content from folder
/ # ^D
```

If multiple folders are mounted for the single bucket on the same machine, only 1 fuse mount will be created. The mount will be shared by docker containers. It will be unmouned when there be no containers to use it.

## License
MIT

[goofys]: https://github.com/kahing/goofys
[volume plugin]: https://docs.docker.com/engine/extend/plugins_volume/
[FUSE]: https://github.com/libfuse/libfuse
[download]: https://github.com/monder/goofys-docker/releases
[AWS auth]: http://docs.aws.amazon.com/sdk-for-go/api/#Configuring_Credentials
[IAM roles]: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html
