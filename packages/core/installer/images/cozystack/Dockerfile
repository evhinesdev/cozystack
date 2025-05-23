FROM golang:alpine3.21 as k8s-await-election-builder

ARG K8S_AWAIT_ELECTION_GITREPO=https://github.com/LINBIT/k8s-await-election
ARG K8S_AWAIT_ELECTION_VERSION=0.4.1

# TARGETARCH is a docker special variable: https://docs.docker.com/engine/reference/builder/#automatic-platform-args-in-the-global-scope
ARG TARGETARCH

RUN apk add --no-cache git make
RUN git clone ${K8S_AWAIT_ELECTION_GITREPO} /usr/local/go/k8s-await-election/ \
 && cd /usr/local/go/k8s-await-election \
 && git reset --hard v${K8S_AWAIT_ELECTION_VERSION} \
 && make \
 && mv ./out/k8s-await-election-${TARGETARCH} /k8s-await-election

FROM golang:alpine3.21 as builder

RUN apk add --no-cache make git
RUN apk add helm --repository=https://dl-cdn.alpinelinux.org/alpine/edge/community

COPY . /src/
WORKDIR /src

RUN go build -o /cozystack-assets-server -ldflags '-extldflags "-static" -w -s' ./cmd/cozystack-assets-server

# Check that versions_map is not changed
RUN make repos

FROM alpine:3.21

RUN apk add --no-cache make
RUN apk add helm kubectl --repository=https://dl-cdn.alpinelinux.org/alpine/edge/community
RUN apk add yq
RUN apk add coreutils

COPY scripts /cozystack/scripts
COPY --from=builder /src/packages/core /cozystack/packages/core
COPY --from=builder /src/packages/system /cozystack/packages/system
COPY --from=builder /src/_out/repos /cozystack/assets/repos
COPY --from=builder /src/_out/logos /cozystack/assets/logos
COPY --from=builder /cozystack-assets-server /usr/bin/cozystack-assets-server
COPY --from=k8s-await-election-builder /k8s-await-election /usr/bin/k8s-await-election
COPY dashboards /cozystack/assets/dashboards

WORKDIR /cozystack
ENTRYPOINT ["/usr/bin/k8s-await-election", "/cozystack/scripts/installer.sh" ]
