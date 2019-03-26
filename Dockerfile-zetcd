FROM golang:alpine
MAINTAINER Eugene Klimov <bloodjazman@gmail.com>
ARG DOCKERIZE_VERSION="v0.4.0"

ENV GOPATH=/zetcd
ENV GOROOT=/usr/local/go

# TODO wait when https://github.com/etcd-io/zetcd/pull/106 will merged
RUN apk --no-cache add git openssl ca-certificates wget && \
    wget -qO- https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz | tar xvz -C /usr/local/bin && \
    git clone https://github.com/kshvakov/zetcd.git /zetcd/src/github.com/etcd-io/zetcd && \
    go build -o /zetcd/bin/zetcd /zetcd/src/github.com/etcd-io/zetcd/cmd/zetcd/zetcd.go && \
    apk del git openssl wget && \
    cp -v /zetcd/bin/zetcd /bin/zetcd && \
    rm -rfv /zetcd

ENV ETCD_HOST=etcd:2379

ENTRYPOINT dockerize -wait tcp://${ETCD_HOST} && zetcd --zkaddr 0.0.0.0:2181 --endpoints ${ETCD_HOST}