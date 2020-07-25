FROM alpine:edge AS build

ENV GOLANG_VERSION dev.go2go
ENV GOLANG_BUILDDIR /usr/src/go
ENV GOPATH /usr/src
ENV GOROOT_FINAL /usr/local/lib/go
ENV GO_LDFLAGS -buildmode=pie

RUN set -eux \
    && apk add --no-cache \
        bash \
        binutils \
        gcc \
        git \
        go \
        musl-dev \
    && export \
        GOOS="$(go env GOOS)" \
        GOARCH="$(go env GOARCH)" \
        GOROOT_BOOTSTRAP="$(go env GOROOT)" \
        GOROOT="${GOLANG_BUILDDIR}" \
        GOBIN="${GOLANG_BUILDDIR}/bin" \
    && case "$(apk --print-arch)" in \
            armhf) export GOARM='6' ;; \
            armv7) export GOARM='7' ;; \
            x86) export GO386='387' ;; \
        esac \
    \
    # Build
    && git clone --depth=1 --single-branch -b ${GOLANG_VERSION} https://github.com/golang/go.git ${GOLANG_BUILDDIR} \
    && cd ${GOLANG_BUILDDIR}/src \
    && ./make.bash -v \
    \
    && export PATH="${GOLANG_BUILDDIR}/bin:$PATH" \
    && go version \
    \
    # Test
    # FIXIT: cgo_test fails due to https://github.com/golang/go/issues/39857
    && ./run.bash -k -no-rebuild -run='!(^cgo_test$)' \
    \
    # Install
    && cd ${GOLANG_BUILDDIR} \
    && mkdir -p ${GOROOT_FINAL}/bin \
    && for binary in go gofmt; do \
            strip bin/${binary}; \
            install -Dm755 bin/${binary} ${GOROOT_FINAL}/bin/${binary}; \
        done \
    && cp -a pkg lib src ${GOROOT_FINAL} \
    && rm -rf ${GOROOT_FINAL}/pkg/obj \
    && rm -rf ${GOROOT_FINAL}/pkg/bootstrap \
    && rm -f ${GOROOT_FINAL}/pkg/tool/*/api \
    && strip ${GOROOT_FINAL}/pkg/tool/$(go env GOOS)_$(go env GOARCH)/* \
    # Remove tests from go/src to reduce package size,
    # these should not be needed at run-time by any program.
    && find ${GOROOT_FINAL}/src -type f -a \( -name "*_test.go" \) \
        -exec rm -rf \{\} \+ \
    && find ${GOROOT_FINAL}/src -type d -a \( -name "testdata" -not -path "*/go2go/*" \) \
        -exec rm -rf \{\} \+ \
    # Remove scripts and docs to reduce package
    && find ${GOROOT_FINAL}/src -type f \
        -a \( -name "*.rc" -o -name "*.bat" -o -name "*.sh" -o -name "Make*" -o -name "README*" \) \
        -exec rm -rf \{\} \+

FROM alpine:edge

# set up nsswitch.conf for Go's "netgo" implementation
# - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
# - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
RUN [ ! -e /etc/nsswitch.conf ] && echo "hosts: files dns" > /etc/nsswitch.conf

ENV GOPATH /go
ENV GOROOT /usr/local/lib/go
ENV PATH "${GOPATH}/bin:${GOROOT}/bin:${PATH}"
ENV GO2PATH "${GOROOT}/src/cmd/go2go/testdata/go2path"

COPY --from=build /usr/local/lib/go /usr/local/lib/go

RUN set -eux \
    && apk add --no-cache ca-certificates \
    && mkdir -p "${GOPATH}/src" "${GOPATH}/bin" \
    && chmod -R 777 "${GOPATH}" \
    && ln -s /usr/local/lib/go/bin/go /usr/bin/ \
    && ln -s /usr/local/lib/go/bin/gofmt /usr/bin/

WORKDIR "${GOPATH}"
