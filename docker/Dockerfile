FROM python:3.7-slim-buster AS builder
ARG AKS_ENGINE_VERSION="v0.44.2"
RUN apt-get update && apt-get install -y ca-certificates curl apt-transport-https lsb-release gnupg unzip
RUN curl -sL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null && \
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ buster main" | tee /etc/apt/sources.list.d/azure-cli.list
RUN cat /etc/apt/sources.list.d/azure-cli.list
RUN apt-get update && apt-get install azure-cli
RUN pip3 wheel ansible[azure]==2.9.1 --wheel-dir=/wheels
RUN curl -s -L --output aks-engine-$AKS_ENGINE_VERSION-linux-amd64.zip https://github.com/Azure/aks-engine/releases/download/$AKS_ENGINE_VERSION/aks-engine-$AKS_ENGINE_VERSION-linux-amd64.zip && \
    unzip aks-engine-$AKS_ENGINE_VERSION-linux-amd64.zip


FROM python:3.7-slim-buster AS runtime
ARG AKS_ENGINE_VERSION="v0.44.2"
ARG KUBECTL_VERSION="v1.16.1"
ENV PYTHONUNBUFFERED 1

COPY --from=builder /wheels /wheels
RUN pip3 install --no-index --find-links=/wheels ansible[azure] && rm -rf /wheels

RUN apt-get update && apt-get install -y ca-certificates curl

COPY --from=builder aks-engine-$AKS_ENGINE_VERSION-linux-amd64/aks-engine /usr/local/bin/aks-engine
RUN curl -s -L --output kubectl https://storage.googleapis.com/kubernetes-release/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl && \
    chmod 755 kubectl && mv kubectl /usr/local/bin

COPY --from=builder /opt/az /opt/az
COPY ./az /usr/bin/az
RUN chmod 755 /usr/bin/az