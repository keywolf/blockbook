FROM ubuntu:18.04

ENV GOLANG_VERSION=go1.17.1.linux-amd64
ENV ROCKSDB_VERSION=v6.22.1
ENV HOME=/home
ENV GOPATH=$HOME/go
ENV PATH=$PATH:$GOPATH/bin
ENV CGO_CFLAGS="-I/home/rocksdb/include"
ENV CGO_LDFLAGS="-L/home/rocksdb -lrocksdb -lstdc++ -lm -lz -lbz2 -lsnappy -llz4 -ldl"

RUN apt-get update && apt-get install -y \
    build-essential git wget pkg-config libzmq3-dev libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev \
    && apt-get clean \
    && cd /home \
    && git clone -b $ROCKSDB_VERSION --depth 1 https://github.com/facebook/rocksdb.git && cd rocksdb \
    && CFLAGS=-fPIC CXXFLAGS=-fPIC make release

RUN wget https://dl.google.com/go/$GOLANG_VERSION.tar.gz && tar xf $GOLANG_VERSION.tar.gz && \
    mv go /opt/go \
    && ln -s /opt/go/bin/go /usr/bin/go \
    && mkdir $HOME/go \

ARG CACHEBUST=1
RUN mkdir -p /home/go/src/github.com  \
    && cd $GOPATH/src \
    && git clone https://github.com/keywolf/blockbook.git \
    && cd blockbook && git checkout v0.3.6 \
#    && git checkout d1fd66597bcd7c0e7db08f5f0215c9ab97033c16 \
#    && dep ensure -vendor-only \
    && go build -tags rocksdb_6_16

WORKDIR /home/go/src/blockbook
COPY blockchaincfg.json /home/go/src/blockbook

ENTRYPOINT ["./blockbook","-datadir=/home/go/src/blockbook/data","-sync","-blockchaincfg=/home/go/src/blockbook/blockchaincfg.json","-internal=:9030","-public=:9130","-certfile=server/testcert","-logtostderr"]
