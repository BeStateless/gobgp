FROM statelesstestregistry.azurecr.io/stateless/base:19 as build

# Install build dependencies.
RUN apt-get update \
 && apt-get install --yes --no-install-recommends \
      autoconf \
      automake \
      build-essential \
      ca-certificates \
      curl \
      golang \
      libreadline-dev \
      texinfo \
      pkg-config \
      bison \
      flex \
      libc-ares-dev \
      python3-dev \
      python3-pytest \
      python3-sphinx \
      libsnmp-dev \
      libcap-dev \
      libjson-c-dev \
      libyang-dev \
      libtool \
      procps \
      socat \
      stgit \
      supervisor \
      unzip \
 && rm -rf /var/lib/apt/lists/*


# Install FRRouting. This simply follows the instructions at
# http://docs.frrouting.org/projects/dev-guide/en/latest/building-frr-for-debian9.html
#
# The version that comes with Debian Buster is quite old. We just need this for Zebra.
RUN addgroup --system --gid 92 frr \
 && addgroup --system --gid 85 frrvty \
 && adduser --system --ingroup frr --home /var/opt/frr --gecos "FRR Suite" --shell /bin/false frr \
 && usermod -a -G frrvty frr \
 && git clone https://github.com/frrouting/frr.git frr \
 && cd frr \
 && git checkout frr-7.1 \
 && ./bootstrap.sh \
 && ./configure \
      --enable-exampledir=/usr/share/doc/frr/examples/ \
      --localstatedir=/var/opt/frr \
      --sbindir=/usr/lib/frr \
      --sysconfdir=/etc/frr \
      --enable-multipath=64 \
      --enable-user=frr \
      --enable-group=frr \
      --enable-vty-group=frrvty \
      --enable-configfile-mask=0640 \
      --enable-logfile-mask=0640 \
      --enable-fpm \
      --with-pkg-git-version \
      --with-pkg-extra-version=-stateless \
 && make -j$(( $(nproc) + 1 )) \
 && make install \
 && install -m 755 -o frr -g frr -d /var/log/frr \
 && install -m 755 -o frr -g frr -d /var/opt/frr \
 && install -m 775 -o frr -g frrvty -d /etc/frr \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/zebra.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/bgpd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/ospfd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/ospf6d.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/isisd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/ripd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/ripngd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/pimd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/ldpd.conf \
 && install -m 640 -o frr -g frr /dev/null /etc/frr/nhrpd.conf \
 && install -m 640 -o frr -g frrvty /dev/null /etc/frr/vtysh.conf \
 && echo include /usr/local/lib >> /etc/ld.so.conf \
 && ldconfig \
 && rm -rf /frr

RUN export PROTOC_ZIP=protoc-3.7.1-linux-x86_64.zip \
 && curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v3.7.1/$PROTOC_ZIP \
 && unzip -o $PROTOC_ZIP -d /usr/local bin/protoc \
 && unzip -o $PROTOC_ZIP -d /usr/local 'include/*' \
 && rm -f $PROTOC_ZIP

# Compile the Stateless-patched GoBGP from source.
WORKDIR /tmp/gobgp
COPY patch ./
COPY patches ./patches
RUN export GOPATH=/go \
 && export PATH="$GOPATH/bin:/usr/local/bin/protoc:/usr/local/go/bin:$PATH" \
 && export GO111MODULE=on \
 && ./patch \
 && mkdir -p "$GOPATH/src" "$GOPATH/bin" \
 && chmod -R 777 "$GOPATH" \
 && mkdir -p $GOPATH/src/github.com/osrg \
 && mv gobgp $GOPATH/src/github.com/osrg \
 && cd $GOPATH/src/github.com/osrg/gobgp \
 && rm -rf /tmp/gobgp \
 && ./tools/grpc/genproto.sh \
 && go mod download \
 && go build -o /usr/local/bin/gobgp ./cmd/gobgp/ \
 && go build -o /usr/local/bin/gobgpd ./cmd/gobgpd/ \
 && rm -rf $GOPATH

COPY ./docker/daemons                /etc/frr
COPY ./docker/frr.conf               /etc/frr
COPY ./docker/supervisor_tenant.conf /etc/supervisor
COPY ./docker/supervisor_link.conf   /etc/supervisor
COPY ./docker/monitor                /usr/local/bin

CMD [ "monitor" ]
