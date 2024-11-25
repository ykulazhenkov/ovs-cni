FROM --platform=$BUILDPLATFORM docker.io/golang:1.23 AS builder

# Support overriding target GOARCH during `make docker-build`
ARG goarch=

# these variable are automatically set during the multiarch build by docker buildx
ARG TARGETOS
ARG TARGETARCH
ENV TARGETOS=${TARGETOS:-linux}
ENV TARGETARCH=${TARGETARCH:-amd64}

ENV GOOS=${TARGETOS}
ENV GOARCH=${goarch:-$TARGETARCH}
ENV CGO_ENABLED=0
ENV GOFLAGS=-mod=vendor

WORKDIR /workdir
RUN mkdir /workdir/bin

COPY go.mod .
COPY go.sum .

RUN go mod download

COPY . .

RUN go build -tags no_openssl -o /workdir/bin/ovs ./cmd/plugin
RUN go build -tags no_openssl -o /workdir/bin/marker ./cmd/marker
RUN go build -tags no_openssl -o /workdir/bin/ovs-mirror-producer ./cmd/mirror-producer
RUN go build -tags no_openssl -o /workdir/bin/ovs-mirror-consumer ./cmd/mirror-consumer

FROM registry.access.redhat.com/ubi9/ubi-minimal

RUN microdnf install -y findutils

COPY --from=builder /workdir/.version /.version
COPY --from=builder /workdir/bin/* /
