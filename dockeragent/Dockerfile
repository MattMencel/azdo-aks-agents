FROM ubuntu:18.04

# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
ENV DEBIAN_FRONTEND=noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

WORKDIR /azp
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  apt-transport-https \
  ca-certificates \
  curl \
  dnsutils \
  docker.io \
  git \
  gnupg \
  iproute2 \
  iputils-ping \
  jq \
  jsonlint \
  libcurl4 \
  libicu60 \
  libunwind8 \
  lsb-release \
  mysql-client \
  netcat \
  nodejs \
  npm \
  openjdk-11-jdk \
  rsync \
  ssh \
  tcpdump \
  unzip \
  xxd \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# INSTALL AZCLI
RUN curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null \
  && AZ_REPO=$(lsb_release -cs) \
  && echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | tee /etc/apt/sources.list.d/azure-cli.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
  azure-cli \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# ADDITIONAL PACKAGES
RUN curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
  && curl -L https://github.com/mikefarah/yq/releases/download/3.3.4/yq_linux_amd64 > /usr/local/bin/yq \
  && chmod a+x /usr/local/bin/docker-compose /usr/local/bin/yq \
  && curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" \
  && chmod +x ./kubectl \
  && mv ./kubectl /usr/local/bin/kubectl \
  && curl -LO https://golang.org/dl/go1.15.2.linux-amd64.tar.gz \
  && tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz

# INSTALL HELM
RUN curl -LO https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz \
  && tar -zxvf helm-v3.3.1-linux-amd64.tar.gz \
  && mv linux-amd64/helm /usr/local/bin/helm

# Add Garden, Go, Helm and Java to PATH
ENV PATH=$PATH:/usr/local/go/bin:/usr/local/bin:/usr/lib/jvm/java-11-openjdk-amd64/bin

RUN mkdir /root/.ssh

COPY ./dockeragent/start.sh .
RUN chmod +x start.sh

CMD ["./start.sh"]
