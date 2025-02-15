ARG ARM
ARG ARCH=${ARM:+arm32v7}
FROM ${ARCH:-amd64}/python:3.11-slim-buster as base

FROM base as builder
ENV PYTHONUNBUFFERED=1
ARG ARM
ARG LIB_ARCH=${ARM:+arm}
ARG MTX_ARCH=${ARM:+armv7}
ARG FFMPEG_ARCH=${ARM:+armv7l}
RUN apt-get update \
    && apt-get install -y tar unzip curl jq g++ git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
COPY requirements.txt /tmp/requirements.txt
RUN pip3 install --disable-pip-version-check --prefix=/build/usr/local -r /tmp/requirements.txt
COPY *.lib /tmp/lib/
RUN mkdir -p /build/app /build/tokens /build/img \
    && curl -L https://github.com/homebridge/ffmpeg-for-homebridge/releases/latest/download/ffmpeg-debian-${FFMPEG_ARCH:-x86_64}.tar.gz \
    | tar xzf - -C /build \
    && MTX_TAG=$(curl -s https://api.github.com/repos/aler9/mediamtx/releases/latest | jq -r .tag_name) \
    && echo -n $MTX_TAG > /build/MTX_TAG \
    && curl -L https://github.com/aler9/mediamtx/releases/download/${MTX_TAG}/mediamtx_${MTX_TAG}_linux_${MTX_ARCH:-amd64}.tar.gz \
    | tar xzf - -C /build/app \
    && cp /tmp/lib/${LIB_ARCH:-amd}.lib /build/usr/local/lib/libIOTCAPIs_ALL.so\
    && rm -rf /tmp/*
COPY . /build/app/

FROM base
ENV PYTHONUNBUFFERED=1 MTX_HLSVARIANT=fmp4 MTX_PROTOCOLS=tcp MTX_READTIMEOUT=20s MTX_LOGLEVEL=warn MTX_WEBRTCICEUDPMUXADDRESS=:8189 SDK_KEY=AQAAAIZ44fijz5pURQiNw4xpEfV9ZysFH8LYBPDxiONQlbLKaDeb7n26TSOPSGHftbRVo25k3uz5of06iGNB4pSfmvsCvm/tTlmML6HKS0vVxZnzEuK95TPGEGt+aE15m6fjtRXQKnUav59VSRHwRj9Z1Kjm1ClfkSPUF5NfUvsb3IAbai0WlzZE1yYCtks7NFRMbTXUMq3bFtNhEERD/7oc504b FLASK_APP=frontend
COPY --from=builder /build /
WORKDIR /app
CMD [ "flask", "run", "--host=0.0.0.0"]