FROM debian:stable-slim as builder

# change /bin/sh to bash from dash
RUN echo "dash dash/sh boolean false" | debconf-set-selections \
    && DEBIAN_FRONTEND=noninteractive dpkg-reconfigure -p low dash || true

# change the apt source to mirror site
RUN sed -ie 's|http://deb|http://ftp.jp|g' /etc/apt/sources.list

RUN apt update && \
    DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends \
        build-essential \
        autogen \
        autoconf \
        flex \
        bison \
        texinfo \
        libncurses5-dev \
        wget \
        unzip \
        zlib1g-dev \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ARG GMP_VER=6.1.0
ARG MPFR_VER=3.1.4
ARG MPC_VER=1.0.3
ARG ISL_VER=0.18
ARG GCC_VER=8.3.0
ARG BINUTILS_VER=2.24
ARG NEWLIB_VER=3.1.0
ARG GCC_MIRROR=http://ftp.tsukuba.wide.ad.jp/software/gcc

ARG RX_ROOT=/root/rx-toolchains
ARG RX_DL=${RX_ROOT}/dl
ARG RX_SRC=${RX_ROOT}/srcs
ARG RX_BUILD=${RX_ROOT}/build
ARG RX_PREFIX=/usr/local/

RUN mkdir -p ${RX_DL} ${RX_SRC} ${RX_BUILD}

#GMP
ARG GMP_URL=${GCC_MIRROR}/infrastructure/gmp-${GMP_VER}.tar.bz2
RUN mkdir -p ${RX_SRC}/gmp ${RX_BUILD}/gmp \
    && wget -P ${RX_DL} ${GMP_URL} \
    && tar xfj ${RX_DL}/gmp-${GMP_VER}.tar.bz2 -C ${RX_SRC}/gmp --strip-components=1 \
    && rm -f ${RX_DL}/gmp-${GMP_VER}.tar.bz2 \
    && cd ${RX_BUILD}/gmp \
    && ${RX_SRC}/gmp/configure \
        --prefix=${RX_PREFIX} \
        --disable-shared \
        --enable-cxx \
    && make -j \
    && make install \
    && cd / \
    && rm -rf ${RX_SRC}/gmp ${RX_BUILD}/gmp

#MPFR
ARG MPFR_URL=${GCC_MIRROR}/infrastructure/mpfr-${MPFR_VER}.tar.bz2
RUN mkdir -p ${RX_SRC}/mpfr ${RX_BUILD}/mpfr \
    && wget -P ${RX_DL} ${MPFR_URL} \
    && tar xfj ${RX_DL}/mpfr-${MPFR_VER}.tar.bz2 -C ${RX_SRC}/mpfr --strip-components=1 \
    && rm -f ${RX_DL}/mpfr-${MPFR_VER}.tar.bz2 \
    && cd ${RX_BUILD}/mpfr \
    && ${RX_SRC}/mpfr/configure \
        --prefix=${RX_PREFIX} \
        --disable-shared \
        --with-gmp=${RX_PREFIX} \
    && make -j all \
    && make install \
    && cd / \
    && rm -rf ${RX_SRC}/mpfr ${RX_BUILD}/mpfr

#MPC
ARG MPC_URL=${GCC_MIRROR}/infrastructure/mpc-${MPC_VER}.tar.gz
RUN mkdir -p ${RX_SRC}/mpc ${RX_BUILD}/mpc \
    && wget -P ${RX_DL} ${MPC_URL} \
    && tar xfz ${RX_DL}/mpc-${MPC_VER}.tar.gz -C ${RX_SRC}/mpc --strip-components=1 \
    && rm -f ${RX_DL}/mpc-${MPC_VER}.tar.gz \
    && cd ${RX_BUILD}/mpc \
    && ${RX_SRC}/mpc/configure \
        --prefix=${RX_PREFIX} \
        --disable-shared \
        --with-gmp=${RX_PREFIX} \
        --with-mpfr=${RX_PREFIX} \
    && make -j \
    && make install \
    && cd / \
    && rm -rf ${RX_SRC}/mpc ${RX_BUILD}/mpc

#ISL
ARG ISL_URL=${GCC_MIRROR}/infrastructure/isl-${ISL_VER}.tar.bz2
RUN mkdir -p ${RX_SRC}/isl ${RX_BUILD}/isl \
    && wget -P ${RX_DL} ${ISL_URL} \
    && tar xfj ${RX_DL}/isl-${ISL_VER}.tar.bz2 -C ${RX_SRC}/isl --strip-components=1 \
    && rm -f ${RX_DL}/isl-${ISL_VER}.tar.bz2 \
    && cd ${RX_BUILD}/isl \
    && ${RX_SRC}/isl/configure \
        --prefix=${RX_PREFIX} \
        --disable-shared \
        --with-gmp-prefix=${RX_PREFIX} \
    && make -j \
    && make install \
    && cd / \
    && rm -rf ${RX_SRC}/isl ${RX_BUILD}/isl

ARG RX_ELF_PREFIX=/usr/local/rx-elf

# binutils
ARG RX_BINUTILS_NAME=binutils_rx_${BINUTILS_VER}_2020q4
ARG RX_BINUTILS_URL=https://llvm-gcc-renesas.com/downloads/d.php?f=rx/binutils/${GCC_VER}.202004-gnurx/${RX_BINUTILS_NAME}.zip
RUN mkdir -p ${RX_BUILD}/binutils \
    && wget -P ${RX_DL} --content-disposition ${RX_BINUTILS_URL} \
    && unzip ${RX_DL}/${RX_BINUTILS_NAME} -d ${RX_SRC} \
    && cd ${RX_BUILD}/binutils \
    && chmod -R +x ${RX_SRC}/${RX_BINUTILS_NAME} \
    && ${RX_SRC}/${RX_BINUTILS_NAME}/configure \
        --prefix=${RX_ELF_PREFIX} \
        --target=rx-elf \
        --with-sysroot=${RX_ELF_PREFIX} \
        --with-target-system-zlib \
        --enable-lto \
        --enable-plugins \
        --disable-nls \
        --disable-werror \
    && make \
    && make install-strip \
    && cd / \
    && rm -rf ${RX_SRC}/${RX_BINUTILS_NAME} ${RX_BUILD}/binutils ${RX_DL}/${RX_BINUTILS_NAME}.zip

ENV PATH=${RX_ELF_PREFIX}/bin:${PATH}

# gcc 1st
ARG RX_GCC_NAME=gcc_rx_${GCC_VER}_2020q4
ARG RX_GCC_URL=https://llvm-gcc-renesas.com/downloads/d.php?f=rx/gcc/${GCC_VER}.202004-gnurx/${RX_GCC_NAME}.zip
RUN mkdir -p ${RX_BUILD}/gcc \
    && wget -P ${RX_DL} --content-disposition ${RX_GCC_URL} \
    && unzip ${RX_DL}/${RX_GCC_NAME} -d ${RX_SRC} \
    && cd ${RX_BUILD}/gcc \
    && chmod -R +x ${RX_SRC}/${RX_GCC_NAME} \
    && ${RX_SRC}/${RX_GCC_NAME}/configure \
        --target=rx-elf \
        --prefix=${RX_ELF_PREFIX} \
        --with-gmp=${RX_PREFIX} \
        --with-mpfr=${RX_PREFIX} \
        --with-mpc=${RX_PREFIX} \
        --with-isl=${RX_PREFIX} \
        --with-newlib \
        --without-headers \
        --enable-initfini-array \
        --disable-libmudflap \
        --disable-nls \
        --disable-tls \
        --disable-shared \
        --disable-multilib \
        --disable-decimal-float \
        --disable-threads \
        --disable-libatomic \
        --disable-libffi \
        --disable-libgomp \
        --disable-libquadmath \
        --disable-libssp \
        --disable-libvtv \
        --disable-libstdcxx \
        --enable-languages=c \
    && make -j all-gcc && make install-gcc \
    && cd / \
    && rm -rf ${RX_BUILD}/gcc ${RX_DL}/${RX_GCC_NAME}.zip

ARG RX_NEWLIB_NAME=newlib_rx_${NEWLIB_VER}_2020q4
ARG RX_NEWLIB_URL=https://llvm-gcc-renesas.com/downloads/d.php?f=rx/newlib/${GCC_VER}.202004-gnurx/${RX_NEWLIB_NAME}.zip

# newlib
RUN mkdir ${RX_BUILD}/newlib \
    && wget -P ${RX_DL} --content-disposition ${RX_NEWLIB_URL} \
    && unzip ${RX_DL}/${RX_NEWLIB_NAME} -d ${RX_SRC} \
    && cd ${RX_BUILD}/newlib \
    && chmod -R +x ${RX_SRC}/${RX_NEWLIB_NAME} \
    && ${RX_SRC}/${RX_NEWLIB_NAME}/configure \
        --target=rx-elf \
        --prefix=${RX_ELF_PREFIX} \
        --with-target-system-zlib \
        --enable-newlib-io-c99-formats \
        --enable-retargetable-locking \
        --enable-newlib-global-atexit \
        --enable-newlib-reent-small \
        --enable-target-optspace \
        --enable-newlib-nano-malloc \
        --enable-lite-exit \
        --enable-newlib-nano-formatted-io \
        --enable-target-optspace \
        --disable-newlib-atexit-dynamic-alloc \
        --disable-newlib-supplied-syscalls \
        --disable-newlib-fvwrite-in-streamio \
        --disable-nls \
        --disable-shared \
    && make -j && make install \
    && find /usr/local/rx-elf/lib -name "*.o" -or -name "*.a" -exec rx-elf-strip --strip-unneeded {} \; \
    && rm -rf ${RX_BUILD}/newlib ${RX_DL}/${RX_NEWLIB_NAME}.zip

RUN ln ${RX_ELF_PREFIX}/rx-elf/lib/libc.a ${RX_ELF_PREFIX}/rx-elf/lib/libc_nano.a \
    && ln ${RX_ELF_PREFIX}/rx-elf/lib/libg.a ${RX_ELF_PREFIX}/rx-elf/lib/libg_nano.a \
    && ln ${RX_ELF_PREFIX}/rx-elf/lib/libm.a ${RX_ELF_PREFIX}/rx-elf/lib/libm_nano.a

# gcc 2nd
RUN mkdir -p ${RX_BUILD}/gcc \
    && cd ${RX_BUILD}/gcc \
    && ${RX_SRC}/${RX_GCC_NAME}/configure \
        --target=rx-elf \
        --prefix=${RX_ELF_PREFIX} \
        --with-gmp=${RX_PREFIX} \
        --with-mpfr=${RX_PREFIX} \
        --with-mpc=${RX_PREFIX} \
        --with-isl=${RX_PREFIX} \
        --with-sysroot=${RX_ELF_PREFIX} \
        --with-target-system-zlib \
        --with-newlib \
        --with-gnu-as \
        --with-gnu-ld \
        --enable-gold \
        --enable-lto \
        --enable-plugins \
        --enable-initfini-array \
        --enable-__cxa_atexit \
        --disable-nls \
        --disable-shared \
        --disable-libgomp \
        --disable-libstdcxx-pch \
        --enable-languages=c,c++ \
    && make -j && make install-strip \
    && cd / \
    && rm -rf ${RX_BUILD}/gcc

FROM debian:stable-slim
RUN apt update && \
    apt install -y --no-install-recommends \
        make \
        git \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*
COPY --from=builder /usr/local/rx-elf /usr/local/rx-elf
ENV PATH=/usr/local/rx-elf/bin:${PATH}
