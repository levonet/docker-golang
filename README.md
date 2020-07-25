# Supported tags and respective `Dockerfile` links

- [`go2go-*`, `go2go-*-buster`, `go2go-buster`,`go2go`, `latest` (*Dockerfile*)](https://github.com/levonet/docker-golang/blob/master/buster/Dockerfile)
- [`go2go-*-alpine`, `go2go-alpine`, `alpine` (*Dockerfile*)](https://github.com/levonet/docker-golang/blob/master/alpine/Dockerfile)

# Docker Image with Go2 from `dev.go2go` branch

<img src="https://raw.githubusercontent.com/docker-library/docs/01c12653951b2fe592c1f93a13b4e289ada0e3a1/golang/logo.png" align="right"/>

This is a nightly build of Golang Docker image from [`dev.go2go` branch](https://github.com/golang/go/tree/dev.go2go).
I hope this build helps the community make more use of the new Go2 features.

# [dev.go2go branch](https://github.com/golang/go/blob/dev.go2go/README.go2go.md)

This branch contains a type checker and a translation tool for
experimentation with generics in Go.
This implements the [generics design draft](https://go.googlesource.com/proposal/+/refs/heads/master/design/go2draft-type-parameters.md).

You can build this branch [as usual](https://golang.org/doc/install/source).
You can then use the new `go2go` tool.
Write your generic code in a file with the extension `.go2` instead of
`.go`.
Run it using `go tool go2go run x.go2`.
There are some sample packages in `cmd/go2go/testdata/go2path/src`.
You can see the full documentation for the tool by running `go doc
cmd/go2go`.

The `go2go` tool will look for `.go2` files using the environment variable `GO2PATH`.
You can find some useful packages that you might want to experiment with by
setting `GO2PATH=$GOROOT/src/cmd/go2go/testdata/go2path`.

If you find bugs in the updated type checker or in the translation
tool, they should be filed in the [standard Go issue
tracker](https://golang.org/issue).
Please start the issue title with `cmd/go2go`.
Note that the issue tracker should only be used for reporting bugs in
the tools, not for discussion of changes to the language.

Unlike typical dev branches, we do not intend any eventual merge of
this code into the master branch.
We will update it for bug fixes and change to the generic design
draft.
If a generics language proposal is accepted, it will be implemented in
the standard compiler, not via a translation tool.
The necessary changes to the go/* packages (go/ast, go/parser, and so
forth) may re-use code from this prototype but will follow the usual
code review process.

# [How to use this image](https://github.com/docker-library/docs/blob/master/golang/content.md#how-to-use-this-image)

**Note:** `/go` is world-writable to allow flexibility in the user which runs the container (for example, in a container started with `--user 1000:1000`, running `go get github.com/example/...` will succeed). While the `777` directory would be insecure on a regular host setup, there are not typically other processes or users inside the container, so this is equivilant to `700` for Docker usage, but allowing for `--user` flexibility.

## Start a Go instance in your app

The most straightforward way to use this image is to use a Go container as both the build and runtime environment. In your `Dockerfile`, writing something along the lines of the following will compile and run your project:

```dockerfile
FROM levonet/golang:go2go

WORKDIR /go/src/app
COPY . .

RUN go get -d -v ./...
RUN go install -v ./...

CMD ["app"]
```

You can then build and run the Docker image:

```console
$ docker build -t my-golang-app .
$ docker run -it --rm --name my-running-app my-golang-app
```

## Compile your app inside the Docker container

There may be occasions where it is not appropriate to run your app inside a container. To compile, but not run your app inside the Docker instance, you can write something like:

```console
$ docker run --rm -v "$PWD":/go/src/myapp -w /go/src/myapp levonet/golang:go2go go build -v
```

Or compile `.go2` source files:

```console
$ docker run --rm -v "$PWD":/go/src/myapp -w /go/src/myapp levonet/golang:go2go go tool go2go build
```

# Image Variants

The `levonet/golang` images come in many flavors, each designed for a specific use case.

## `levonet/golang:<version>`

This is the defacto image. If you are unsure about what your needs are, you probably want to use this one.
It is designed to be used both as a throw away container (mount your source code and start the container to start your app), as well as the base to build other images off of.

Some of these tags may have names like buster in them. These are the suite code names for releases of [Debian](https://wiki.debian.org/DebianReleases) and indicate which release the image is based on. If your image needs to install any additional packages beyond what comes with the image, you'll likely want to specify one of these explicitly to minimize breakage when there are new releases of Debian.

## `levonet/golang:<version>-alpine`

This image is based on the popular [Alpine Linux project](https://alpinelinux.org), available in [the `alpine` official image](https://hub.docker.com/_/alpine). Alpine Linux is much smaller than most distribution base images (~5MB), and thus leads to much slimmer images in general.

This variant is highly experimental, and *not* officially supported by the Go project (see [golang/go#19938](https://github.com/golang/go/issues/19938) for details).

The main caveat to note is that it does use [musl libc](https://musl.libc.org) instead of [glibc and friends](https://www.etalabs.net/compare_libcs.html), which can lead to unexpected behavior. See [this Hacker News comment thread](https://news.ycombinator.com/item?id=10782897) for more discussion of the issues that might arise and some pro/con comparisons of using Alpine-based images.

To minimize image size, additional related tools (such as `git`, `gcc`, or `bash`) are not included in Alpine-based images. Using this image as a base, add the things you need in your own Dockerfile (see the [`alpine` image description](https://hub.docker.com/_/alpine/) for examples of how to install packages if you are unfamiliar). See also [docker-library/golang#250 (comment)](https://github.com/docker-library/golang/issues/250#issuecomment-451201761) for a longer explanation.

## License

View [license information](https://golang.org/LICENSE) for the software contained in this image or [license information](https://github.com/levonet/docker-golang/blob/master/LICENSE) for the Golang Dockerfile.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

Some additional license information which was able to be auto-detected might be found in [the `repo-info` repository's `golang/` directory](https://github.com/docker-library/repo-info/tree/master/repos/golang).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
