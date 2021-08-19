FROM debian:bullseye-slim
MAINTAINER Brett Viren <brett.viren@gmail.com>

ARG WCTOOLKIT_VERSION=0.16.0
ARG WCPYTHON_VERSION=0.14.0
ARG JSONNET_VERSION=0.17.0
# verions for boost, tbb, etc?

LABEL \
    wctoolkit_version=$WCTOOLKIT_VERSION \
    wcpython_version=$WCPYTHON_VERSION \
    jsonnet_version=$JSONNET_VERSION    

COPY packages packages

RUN apt-get update -qq \
    && ln -sf /usr/share/zoneinfo/UTC /etc/localtime \
    && apt-get -y install $(cat /packages) \
    && apt-get clean \
    && rm -rf /usr/share/locale/[a-df-z]* \
    && rm -f /packages

Run mkdir -p /usr/local/{bin,lib,include



# RUN cd /root && \
#     git clone https://github.com/google/jsonnet.git && \
#     cd jsonnet && \
#     git checkout v${JSONNET_VERSION} && \
#     make -j$(nproc) && make install && \
#     cd /root && \
#     rm -rf jsonnet /usr/local/lib/libjsonnet++.so* /usr/local/include/libjsonnet++.h

RUN cd /root && \
    wget -O /usr/include/libjsonnet.h \
      https://raw.githubusercontent.com/google/jsonnet/v${JSONNET_VERSION}/include/libjsonnet.h && \
    git clone https://github.com/google/go-jsonnet.git && \
    cd go-jsonnet && \
    git checkout v${JSONNET_VERSION} && \
    go build ./cmd/jsonnet && \
    go build ./cmd/jsonnetfmt && \
    go build ./cmd/jsonnet-deps && \
    CGO_CXXFLAGS="-std=c++17 -Wall -I/usr/local/include" \
      go build -o libgojsonnet.so -buildmode=c-shared ./c-bindings && \
    cp libgojsonnet.h /usr/local/include/ && \
    cp libgojsonnet.so /usr/local/lib/ && \
    cp jsonnet jsonnetfmt jsonnet-deps /usr/local/bin/ && \
    cd /root && \
    rm -rf go-jsonnet

RUN cd /root && \
    wget https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.tar.bz2 && \
    tar -xf boost_1_76_0.tar.bz2 && \
    cd /root/boost_1_76_0/ && \
    ./bootstrap.sh --prefix=/usr/local && \
    sed -i -e 's,using python : 2.7 : "/usr" ;,using python : 3.7 : "/usr" ;,' project-config.jam && \
    ./b2 install && \
    cd /root && rm -rf boost_1_76_0*

RUN mkdir -p /opt && cd /opt && \
    wget https://github.com/oneapi-src/oneTBB/releases/download/v2021.1.1/oneapi-tbb-2021.1.1-lin.tgz && \
    tar -xf oneapi-tbb-2021.1.1-lin.tgz && \
    rm -rf oneapi-tbb-2021.1.1-lin.tgz oneapi-tbb-2021.1.1/lib/ia32

RUN git clone https://github.com/gabime/spdlog.git /root/spdlog \
    && mkdir /root/spdlog/build \
    && cd /root/spdlog/build \
    && cmake .. -DSPDLOG_BUILD_SHARED=yes && make -j install \
    && cd /root && rm -rf spdlog

RUN cd /root \
    && git clone https://github.com/WireCell/wire-cell-toolkit.git \
    && cd wire-cell-toolkit \
    && git checkout ${WCTOOLKIT_VERSION}

RUN ln -sf /usr/bin/python3 /usr/local/bin/python
RUN cd /root/wire-cell-toolkit \
    && ./wcb configure --prefix=/usr/local \
    --with-jsonnet=/usr/local \
    --with-jsonnet-libs=gojsonnet \
    --with-tbb=/opt/oneapi-tbb-2021.1.1 \
    --with-tbb-include=/opt/oneapi-tbb-2021.1.1/include \
    --with-tbb-lib=/opt/oneapi-tbb-2021.1.1/lib/intel64/gcc4.8 \
    --boost-include=/usr/local/include \
    --boost-libs=/usr/local/lib --boost-mt \
    && ./wcb --notests install \
    && rm -rf /usr/local/share/wirecell \
    && cp -a cfg /usr/local/share/wirecell \
    && cd /root && rm -rf wire-cell-toolkit

# prefer pip's snakemake as debian's brings in a crapload of node.js
RUN pip install snakemake ipython matplotlib numpy jupyterlab \
    && rm -rf /root/.cache
# fixme: need to make some release!
RUN pip install -e git+https://github.com/wirecell/wire-cell-python.git@${WCPYTHON_VERSION}#egg=wirecell \
    && rm -rf /root/.cache

ENV     HOME /home/user

RUN     useradd \
            --create-home \
            --home-dir $HOME \
            --uid 1000 \
            user \
 &&     chown -R user:user $HOME

WORKDIR $HOME

ENV WIRECELL_PATH=/usr/local/share/wirecell

ENTRYPOINT ["/bin/bash","-l","-c"]
CMD "bash -l"


