# Klar
Integration of Clair and Docker Registry

Klar is a simple tool to analyze images stored in a private or public  Docker registry for security vulnerabilities using Clair https://github.com/coreos/clair. Klar is designed to be used as an integration tool so it relies on enviroment variables. It's a single binary which requires no dependencies.

## Binary installation

The simples way is to download the latest relase (for OSX and Linux) from https://github.com/optiopay/klar/releases/ and put the binary in a folder in your `PATH`, make sure it has execute permission

## Installation from source code

Make sure you have Go language compiler installed and configured https://golang.org/doc/install

Then run

    go get github.com/optiopay/klar

make sure your Go binary folder is in your `PATH` (e.g. `export PATH=$PATH:/usr/local/go/bin`)


## Usage

Klar process returns `0` if number of detected high severity vulnerabilities in an image is less or equals than threshold (see below), otherwise it returns `1`.

Klar can be configured via the following environment variables:

* `CLAIR_ADDR` - address of Clair server, the most complete form is `protocol://host:port`
protocol and port may be omited, `http` and `6060` are used by default

* `CLAIR_OUTPUT` - severity level threshold, vulnerabilities with severity level higher than or equal to this threshold
will be outputed. Supported levels are `Unknown`, `Negligible`, `Low`, `Medium`, `High`, `Critical`, `Defcon1`.
Default is `Unknown`.

* `CLAIR_THRESHOLD` - how many high severity vulnerabilities Klar can tolerate before returning `1`. Default is 0.

* `DOCKER_USER` - Docker registry account name

* `DOCKER_PASSWORD` - Docker registry account password

* `DOCKER_INSECURE` - Allow Klar to access registries with bad SSL certificates. Default is `false`. Clair will 
need to be booted with `-insecure-tls` for this to work.

* `REGISTRY_INSECURE` - Allow Klar to access insecure registries (HTTP only). Default is `false`.

* `JSON_OUTPUT` - Output JSON, not plain text. Default is `false`.

Usage:

    CLAIR_ADDR=http://localhost CLAIR_OUTPUT=High CLAIR_THRESHOLD=10 DOCKER_USER=me DOCKER_PASSWORD=secret klar postgres:9.5.1

## Dockerized version

Klar can be dockerized. Go to `$GOPATH/src/github.com/optiopay/klar` and build Klar in project root:

    go build .

If you are on Mac don't forget to build it for Linux:

    GOOS=linux go build .

To build Docker image run in the project root (replace `klar` with fully qualified name if you like):

    docker build -t klar .

Then create an env file or pass env vars as separate ``--env` arguments. For example save it as `my-klar.env`

    CLAIR_ADDR=http://localhost
    CLAIR_OUTPUT=High
    CLAIR_THRESHOLD=10
    DOCKER_USER=me
    DOCKER_PASSWORD=secret

Then run

    docker run --env-file=my-klar-env klar postgres:9.5.1

## Amazon ECR support
There is no permanent username/password for Amazon ECR, the credentials must be retrived using `aws ecr get-login` and they are valid for 12 hours. Here is a sample script which may be used to provide Klar with ECR credentials:

    DOCKER_LOGIN=`aws ecr get-login`
    PASSWORD=`echo $DOCKER_LOGIN | cut -d' ' -f6`
    REGISTRY=`echo $DOCKER_LOGIN | cut -d' ' -f9 | sed "s/https:\/\///"`
    DOCKER_USER=AWS DOCKER_PASSWORD=${PASSWORD} ./klar ${REGISTRY}/my-image
