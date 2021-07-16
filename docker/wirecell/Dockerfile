FROM debian:bullseye-slim
MAINTAINER Brett Viren <brett.viren@gmail.com>

COPY packages packages

RUN apt-get update -qq \
 && ln -sf /usr/share/zoneinfo/UTC /etc/localtime \
 && apt-get -y install $(cat /packages) \
 && rm -f /packages \
 && apt-get clean \
 && rm -rf /usr/share/locale/[a-df-z]* 

RUN cd /root && \
    git clone https://github.com/google/jsonnet.git && \
    cd jsonnet && \
    git checkout v0.17.0 && \
    make -j$(nproc) && make install && \
    cd /root && \
    rm -rf jsonnet
# fixme: add go-jsonnet

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
    && git checkout 0.15.0

RUN ln -sf /usr/bin/python3 /usr/local/bin/python
RUN cd /root/wire-cell-toolkit \
    && ./wcb configure --prefix=/usr/local \
    --with-jsonnet=/usr/local \
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
RUN pip install -e git+https://github.com/wirecell/wire-cell-python.git@master#egg=wirecell \
    && rm -rf /root/.cache



ENTRYPOINT ["/bin/bash"]
