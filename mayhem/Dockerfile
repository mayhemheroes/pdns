# Build Stage
FROM --platform=linux/amd64 ubuntu:22.04 AS builder

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        clang \
        autoconf \
        automake \
        bison \
        dh-autoreconf \
        flex \
        libboost-all-dev \
        libluajit-5.1-dev \
        libedit-dev \
        libprotobuf-dev \
        libssl-dev \
        libtool \
        make \
        pkg-config \
        protobuf-compiler \
        ragel

COPY . /src/pdns
RUN rm -rf /src/pdns/mayhem
WORKDIR /src/pdns

ENV CXX=clang++ CC=clang
ENV CXXFLAGS="-fsanitize=fuzzer-no-link -O1"
ENV LIB_FUZZING_ENGINE="-fsanitize=fuzzer"
ENV OUT=/out
ENV SRC=/src

RUN sed -i "s/BOOST_PROGRAM_OPTIONS(\[mt\])//" configure.ac && \
    sed -i "s/AC_MSG_ERROR(\[Boost Program Options library not found\])/AC_MSG_NOTICE([Boost Program Options library not found])/" configure.ac && \
    sed -i "s/AC_CC_PIE//" configure.ac && \
    autoreconf -vi && \
    ./configure \
        --without-dynmodules \
        --with-modules="" \
        --disable-lua-records \
        --disable-ixfrdist \
        --enable-fuzz-targets \
        --disable-dependency-tracking \
        --disable-silent-rules || true && \
    cd pdns && \
    make -j$(nproc) fuzz_targets

RUN mkdir -p /out && cp /src/pdns/pdns/fuzz_target_* /out/

# Package Stage
FROM --platform=linux/amd64 ubuntu:22.04

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        libboost-system1.74.0 \
        libssl3 \
        libluajit-5.1-2 \
        libedit2 \
        libprotobuf23

COPY --from=builder /out/fuzz_target_dnsdistcache /out/fuzz_target_dnsdistcache
