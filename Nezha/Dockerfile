FROM alpine:3.20

LABEL maintainer="nezha"

ARG VERSION=latest
ENV TZ=Asia/Shanghai
ENV LANG=zh_CN.UTF-8

WORKDIR /dashboard

RUN apk add --no-cache ca-certificates tzdata curl unzip \
 && mkdir -p /dashboard/data \
 && if [ "$VERSION" = "latest" ]; then \
      URL="https://github.com/nezhahq/nezha/releases/latest/download/dashboard-linux-amd64.zip"; \
    else \
      URL="https://github.com/nezhahq/nezha/releases/download/${VERSION}/dashboard-linux-amd64.zip"; \
    fi \
 && curl -fL -o /tmp/dashboard.zip "$URL" \
 && unzip /tmp/dashboard.zip -d /tmp/dashboard-bin \
 && mv /tmp/dashboard-bin/dashboard-linux-amd64 /dashboard/app \
 && chmod +x /dashboard/app \
 && rm -rf /tmp/dashboard.zip /tmp/dashboard-bin \
 && apk del curl unzip \
 && printf '%s\n' \
      '#!/bin/sh' \
      'cd /dashboard || exit 1' \
      'exec /dashboard/app' \
      > /start.sh \
 && chmod +x /start.sh

VOLUME ["/dashboard/data"]

EXPOSE 8008

CMD ["/start.sh"]
