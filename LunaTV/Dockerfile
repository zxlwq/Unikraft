FROM ghcr.io/moontechlab/lunatv:latest

LABEL maintainer="lunatv"

ENV NODE_ENV=production
ENV HOSTNAME=0.0.0.0
ENV PORT=3000
ENV DOCKER_ENV=true
ENV TZ=Asia/Shanghai
ENV PATH="/usr/local/bin:/usr/bin:/bin"

WORKDIR /app

EXPOSE 3000

CMD ["/usr/local/bin/node", "/app/start.js"]
