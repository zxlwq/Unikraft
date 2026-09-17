FROM debian:bookworm-slim

LABEL maintainer="openlist"

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

RUN wget -O openlist.tar.gz https://github.com/OpenListTeam/OpenList/releases/latest/download/openlist-linux-amd64.tar.gz \
    && tar -zxvf openlist.tar.gz \
    && rm openlist.tar.gz \
    && chmod +x openlist 

RUN mkdir -p /app/data

VOLUME ["/app/data"]

EXPOSE 5244

ENTRYPOINT ["/openlist"]

CMD ["openlist", "server"]
