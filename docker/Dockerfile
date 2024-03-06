FROM ubuntu:latest

RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    apt-transport-https ca-certificates curl software-properties-common \
    bash fish zsh \
    build-essential gdb gcc \
    emacs vim nano \
    libncursesw5 libncurses5-dev \
    qemu qemu-system \
    wget file expect git tmux htop \
    python3 python3-pip lsb-release gnupg bear \
    bear clang clangd clang-format clang-tidy

ADD pintos.tar /usr/class/cs140/pintos
COPY bochs-2.6.11.tar.gz /usr/class/cs140/pintos/pintos/src/misc

RUN cd /usr/class/cs140/pintos/pintos/src/misc && \
    sh ./bochs-2.6.11-build.sh /usr/local && \
    rm -f bochs-2.6.11.tar.gz && \
    cp /usr/class/cs140/pintos/pintos/src/utils/backtrace /usr/local/bin && \
    cp /usr/class/cs140/pintos/pintos/src/utils/pintos-gdb /usr/local/bin && \
    cp /usr/class/cs140/pintos/pintos/src/utils/pintos /usr/local/bin && \
    cp /usr/class/cs140/pintos/pintos/src/utils/Pintos.pm /usr/local/bin && \
    cp /usr/class/cs140/pintos/pintos/src/utils/pintos-set-cmdline /usr/local/bin && \
    cp /usr/class/cs140/pintos/pintos/src/utils/pintos-mkdisk /usr/local/bin

RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    openssh-client uuid-runtime

CMD ["bash"]