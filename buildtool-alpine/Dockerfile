# The docker image to build the go binaries
FROM golang:1.10.3-alpine3.7
MAINTAINER Uladzimir Trehubenka <utrehubenka@infoblox.com>
WORKDIR /tmp

# Install build tools
ENV PKG bash curl gcc git linux-headers make musl-dev openssh protobuf unzip

RUN apk update --no-cache && \
    apk upgrade --no-cache && \
    apk add --no-cache $PKG

# The version and the binaries checksum for the glide package manager
ENV GLIDE_VERSION 0.13.1
ENV GLIDE_DOWNLOAD_URL https://github.com/Masterminds/glide/releases/download/v${GLIDE_VERSION}/glide-v${GLIDE_VERSION}-linux-amd64.tar.gz
ENV GLIDE_DOWNLOAD_SHA256 c403933503ea40308ecfadcff581ff0dc3190c57958808bb9eed016f13f6f32c

# Download and install the glide package manager.
RUN curl -fsSL ${GLIDE_DOWNLOAD_URL} -o glide.tar.gz \
    && echo "${GLIDE_DOWNLOAD_SHA256}  glide.tar.gz" | sha256sum -c - \
    && tar -xzf glide.tar.gz --strip-components=1 -C /usr/local/bin \
    && rm -rf glide.tar.gz

# Install as the protoc plugins as build-time dependencies
COPY glide.yaml .

# Compile binaries for the protocol buffer plugins. We need specific
# versions of these tools, this is why we at first step install glide,
# download required versions and then installing them.
RUN glide up --strip-vendor --skip-test \
    && cp -r vendor/* ${GOPATH}/src/ \
    && go install github.com/golang/protobuf/protoc-gen-go \
    && go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger \
    && go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway \
    && rm -rf vendor/* ${GOPATH}/pkg/* ${GOPATH}/src/*

# Install the Go linter and Go junit binary
RUN go get github.com/golang/lint/golint \
    && go get github.com/jstemmer/go-junit-report \
    && rm -rf ${GOPATH}/pkg/* ${GOPATH}/src/*

WORKDIR ${GOPATH}
