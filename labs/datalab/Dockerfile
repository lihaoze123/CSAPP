FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    gcc \
    gcc-multilib \
    make \
    perl \
    libc6-dev-i386 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /datalab

RUN chmod +x /datalab/dlc /datalab/driver.pl 2>/dev/null || true

CMD ["/bin/bash"]
