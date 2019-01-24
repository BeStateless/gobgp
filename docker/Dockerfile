FROM golang:stretch
COPY ./docker/monitor /
WORKDIR /tmp/gobgp
COPY patch ./
COPY patches ./patches
RUN apt-get update \
 && apt-get install --yes --no-install-recommends \
      socat \
      stgit \
 && ./patch \
 && mkdir -p $GOPATH/src/github.com/osrg \
 && mv gobgp $GOPATH/src/github.com/osrg \
 && cd $GOPATH/src/github.com/osrg/gobgp \
 && rm -rf /tmp/gobgp \
 && go get -u github.com/golang/dep/cmd/dep \
 && dep ensure \
 && go build -o /gobgp ./cmd/gobgp/ \
 && go build -o /gobgpd ./cmd/gobgpd/ \
 && rm -rf $GOPATH
WORKDIR /
CMD [ "/monitor" ]
