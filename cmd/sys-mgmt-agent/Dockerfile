#
# Copyright (c) 2018
# Cavium
#
# SPDX-License-Identifier: Apache-2.0
#

FROM golang:1.12-alpine AS builder

ENV GO111MODULE=on
WORKDIR /go/src/github.com/edgexfoundry/edgex-go

# The main mirrors are giving us timeout issues on builds periodically.
# So we can try these.
RUN sed -e 's/dl-cdn[.]alpinelinux.org/nl.alpinelinux.org/g' -i~ /etc/apk/repositories

RUN apk update && apk add make bash git
COPY go.mod .
# COPY go.sum .

RUN go mod download

COPY . .

# Build the SMA executable.
RUN make cmd/sys-mgmt-agent/sys-mgmt-agent

# Build the golang "executor" executable.
RUN make cmd/sys-mgmt-executor/sys-mgmt-executor

# Get the Docker-in-Docker image layered-in now.
FROM docker:latest

RUN apk add --no-cache bash
RUN rm -rf /var/cache/apk/*

LABEL license='SPDX-License-Identifier: Apache-2.0' \
      copyright='Copyright (c) 2017-2019: Mainflux, Cavium, Dell'

# Copy over the SMA executable bits.
COPY --from=builder /go/src/github.com/edgexfoundry/edgex-go/cmd/sys-mgmt-agent/sys-mgmt-agent /
COPY --from=builder /go/src/github.com/edgexfoundry/edgex-go/cmd/sys-mgmt-agent/res/docker/configuration.toml /res/docker/configuration.toml
COPY --from=builder /go/src/github.com/edgexfoundry/edgex-go/cmd/sys-mgmt-agent/res/docker/configuration.toml /res/configuration.toml

# Copy over the golang "executor" executable.
COPY --from=builder /go/src/github.com/edgexfoundry/edgex-go/cmd/sys-mgmt-executor/sys-mgmt-executor /

RUN apk --no-cache add py-pip
RUN pip install docker-compose==1.23.2
RUN apk --no-cache add curl

ENTRYPOINT ["/sys-mgmt-agent","-cp=consul.http://edgex-core-consul:8500","--registry","--profile=docker","--confdir=/res"]

# Following (commented-out) line is a life-saving DEBUG statement.
# ENTRYPOINT tail -f /dev/null
