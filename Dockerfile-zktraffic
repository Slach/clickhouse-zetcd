FROM alpine:latest
MAINTAINER Eugene Klimov <bloodjazman@gmail.com>

RUN apk --no-cache add python py-pip py-psutil && \
    pip install --no-cache-dir zktraffic && \
    apk del py-pip

ENTRYPOINT /usr/bin/zk-dump