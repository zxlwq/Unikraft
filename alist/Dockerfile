FROM debian:bookworm-slim

LABEL maintainer="alist"

ENV TZ=Asia/Shanghai
ENV PATH="/root/.local/bin:~/.local/bin:$PATH"

WORKDIR /app

RUN apt-get update && apt-get install -y \
    wget \
    curl \
    ca-certificates \
    tzdata \
    unzip \
    bash \
    && rm -rf /var/lib/apt/lists/*

ARG VERSION=latest

RUN wget -O alist.tar.gz https://github.com/ykxVK8yL5L/alist/releases/download/${VERSION}/alist-linux-amd64.tar.gz \
    && tar -zxvf alist.tar.gz \
    && rm alist.tar.gz \
    && chmod +x alist 

RUN mkdir -p /app/data

VOLUME ["/app/data"]

EXPOSE 5244

ENTRYPOINT ["/alist"]

CMD ["alist", "server"]
