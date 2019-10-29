FROM statelesstestregistry.azurecr.io/stateless/base:8 as build

# Install build dependencies.
RUN apt-get update \
 && apt-get install --yes --no-install-recommends \
      build-essential \
      ca-certificates \
      golang \
      procps \
      socat \
      stgit \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/gobgp
COPY patch ./
COPY patches ./patches
RUN export GOPATH=/go \
 && export PATH="$GOPATH/bin:/usr/local/go/bin:$PATH" \
 && export GO111MODULE=on \
 && ./patch \
 && mkdir -p "$GOPATH/src" "$GOPATH/bin" \
 && chmod -R 777 "$GOPATH" \
 && mkdir -p $GOPATH/src/github.com/osrg \
 && mv gobgp $GOPATH/src/github.com/osrg \
 && cd $GOPATH/src/github.com/osrg/gobgp \
 && rm -rf /tmp/gobgp \
 && go mod download \
 && go build -o /gobgp ./cmd/gobgp/ \
 && go build -o /gobgpd ./cmd/gobgpd/ \
 && rm -rf $GOPATH

COPY ./docker/monitor /
CMD [ "/monitor" ]
