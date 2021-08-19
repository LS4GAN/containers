FROM ls4gan/wirecell:0.16.0
MAINTAINER Brett Viren <brett.viren@gmail.com>

ARG TOYZERO_VERSION=0.3.0

USER user

RUN cd \
    && git clone https://github.com/ls4gan/toyzero.git \
    && cd toyzero \
    && git checkout ${TOYZERO_VERSION}



