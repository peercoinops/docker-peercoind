FROM debian:latest
MAINTAINER peerchemist <peerchemist@protonmail.ch> (@peerboxPPC)

ENV PPC_VERSION=0.5.4
ENV PPC_SHA256 61d9de36f111c833afa491d4db2473985e126a26fd5f913db6663bdec88c1075
ENV GOSU_VERSION=1.9

## dependencies
RUN apt-get update -y \
  && apt-get install --no-install-recommends -y curl ca-certificates git-core build-essential libssl-dev libdb++-dev \
  libboost-program-options-dev libboost-filesystem-dev libboost-system-dev libboost-thread-dev

## gosu
RUN gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4
RUN curl -o /usr/local/bin/gosu -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture) \
	&& curl -o /usr/local/bin/gosu.asc -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture).asc \
	&& gpg --verify /usr/local/bin/gosu.asc \
	&& rm /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu

## peercoin
RUN useradd -r sunny

RUN curl -SLO4 https://github.com/ppcoin/ppcoin/archive/v${PPC_VERSION}ppc.tar.gz \
  && tar xf v${PPC_VERSION}ppc.tar.gz \
  && echo "$PPC_SHA256 v${PPC_VERSION}ppc.tar.gz" | sha256sum -c - \
  && cd ppcoin-${PPC_VERSION}ppc \
  ## make ppcoind
  && make -f makefile.unix USE_UPNP= -e PIE=1 -C src \
  && install -Dm755 "src/ppcoind" "/usr/local/bin/"

## cleanup
RUN rm v${PPC_VERSION}ppc.tar.gz && rm -rf ppcoin-${PPC_VERSION}ppc
RUN apt-get remove -y --purge curl ca-certificates git-core build-essential libssl-dev libdb++-dev \
  libboost-program-options-dev libboost-filesystem-dev libboost-system-dev libboost-thread-dev \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PPC_DATA=/home/sunny/.ppcoind \
  PATH=/usr/local/bin/:$PATH

VOLUME ["/home/sunny/.ppcoind"]

## docker
COPY docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 9901 9902

CMD ["ppcoind"]