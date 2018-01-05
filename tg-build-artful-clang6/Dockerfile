# Development image for Typegring with clang-6

FROM ubuntu:17.10
MAINTAINER Zsolt Parragi <zsolt.parragi@cancellar.hu>

RUN DEBIAN_FRONTEND=noninteractive\
    apt-get update && \
    apt-get install -y software-properties-common wget && \
    wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|apt-key add && \
    apt-add-repository "deb http://apt.llvm.org/artful/ llvm-toolchain-artful main" && \
    apt-get update && \
    apt-get -y upgrade && \
    apt-get install -y \
            clang-6.0 clang-6.0-doc libclang-common-6.0-dev libclang-6.0-dev libclang1-6.0 libclang1-6.0-dbg libllvm6.0 libllvm6.0-dbg lldb-6.0 llvm-6.0 llvm-6.0-dev llvm-6.0-doc llvm-6.0-examples llvm-6.0-runtime clang-format-6.0 python-clang-6.0 liblldb-6.0-dbg lld-6.0 libfuzzer-6.0-dev \
            build-essential gdb \
            zlib1g-dev \
            cmake ninja-build curl subversion git &&\
    rm -rf /var/lib/apt/lists/* 

# FixUID
RUN addgroup --gid 1000 docker && \
    adduser --uid 1000 --ingroup docker --home /home/docker --shell /bin/sh --disabled-password --gecos "" docker

RUN USER=docker && \
    GROUP=docker && \
    curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.2/fixuid-0.2-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - && \
    chown root:root /usr/local/bin/fixuid && \
    chmod 4755 /usr/local/bin/fixuid && \
    mkdir -p /etc/fixuid && \
    printf "user: $USER\ngroup: $GROUP\n" > /etc/fixuid/config.yml

RUN \
    mkdir /work && \
    chown -R docker:docker /work

USER docker:docker

# add an msanified libcxx for testing with msan
RUN mkdir /work/libcxx-msan-trunk/ && \
    cd /work/libcxx-msan-trunk/ && \
    svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm && \
    (cd llvm/projects && svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx) && \
    (cd llvm/projects && svn co http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi) && \
    mkdir -p build && cd build && \
    CC=clang-6.0 CXX=clang++-6.0 cmake ../llvm -DCMAKE_INSTALL_PREFIX=.. -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_SANITIZER=Memory && \
    make install-libcxx install-libcxxabi

ENTRYPOINT ["fixuid"]

