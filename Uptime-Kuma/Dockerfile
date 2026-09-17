FROM node:22-alpine3.20 AS node

FROM node:22-alpine3.20 AS build

WORKDIR /app

RUN apk add --no-cache python3 make g++ git ca-certificates curl

ARG VERSION=2.5.4

RUN git clone --depth 1 --branch "${VERSION}" https://github.com/louislam/uptime-kuma.git .

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1
ENV NODE_ENV=production

RUN npm ci --omit=dev --no-audit \
 && curl -fL -o dist.tar.gz \
      "https://github.com/louislam/uptime-kuma/releases/download/${VERSION}/dist.tar.gz" \
 && tar -xzf dist.tar.gz \
 && rm -f dist.tar.gz \
 && test -f dist/index.html \
 && npm cache clean --force \
 && stub() { \
      pkg="$1"; shift; \
      rm -rf "node_modules/$pkg"; \
      mkdir -p "node_modules/$pkg"; \
      printf '%s\n' "$@" > "node_modules/$pkg/index.js"; \
      printf '{"name":"%s","main":"index.js","version":"0.0.0-stub"}\n' "$pkg" \
        > "node_modules/$pkg/package.json"; \
    }; \
    stub playwright-core \
      'module.exports={chromium:{launch:async()=>{throw new Error("unavailable")},connect:async()=>{throw new Error("unavailable")}}};'; \
    stub oracledb \
      'module.exports={OUT_FORMAT_OBJECT:4002,getConnection:async()=>{throw new Error("unavailable")}};'; \
    stub mongodb \
      'class MongoClient{constructor(){throw new Error("unavailable")}static connect(){return Promise.reject(new Error("unavailable"))}};module.exports={MongoClient};'; \
    stub mssql \
      'module.exports={connect:async()=>{throw new Error("unavailable")},query:async()=>{throw new Error("unavailable")}};'; \
    stub gamedig \
      'module.exports={GameDig:{query:async()=>{throw new Error("unavailable")}}};'; \
    rm -rf node_modules/@grpc/grpc-js \
 && mkdir -p node_modules/@grpc/grpc-js \
 && printf '%s\n' 'module.exports={credentials:{},loadPackageDefinition:()=>({}),makeGenericClientConstructor:()=>function(){}};' \
      > node_modules/@grpc/grpc-js/index.js \
 && printf '%s\n' '{"name":"@grpc/grpc-js","main":"index.js","version":"0.0.0-stub"}' \
      > node_modules/@grpc/grpc-js/package.json \
 && rm -rf .git test docker .github \
      node_modules/@types \
      node_modules/typescript \
      node_modules/cpu-features \
      node_modules/ssh2/util/pagent.exe \
 && find node_modules -type f \( \
      -name "*.md" -o -name "*.map" -o -name "*.ts" -o -name "*.d.ts" \
      -o -name "LICENSE*" -o -name "CHANGELOG*" -o -name "*.min.js.map" \
      -o -name "*.exe" -o -name "*.a" -o -name "*.lib" \
    \) -delete 2>/dev/null || true \
 && find node_modules -type d \( \
      -name docs -o -name example -o -name examples -o -name test -o -name tests \
      -o -name __tests__ -o -name .github -o -name coverage \
    \) -prune -exec rm -rf {} + 2>/dev/null || true \
 && mkdir -p /app/data \
 && printf '%s\n' \
      "process.chdir('/app');" \
      "require('./server/server.js');" \
      > /app/start.js

FROM alpine:3.20 AS sys

RUN set -xe; \
    mkdir -p /target/etc /blank; \
    apk --no-cache add ca-certificates tzdata; \
    update-ca-certificates; \
    cp /usr/share/zoneinfo/Asia/Shanghai /target/etc/localtime; \
    echo "Asia/Shanghai" > /target/etc/timezone

FROM scratch

ENV TZ=Asia/Shanghai
ENV UPTIME_KUMA_IS_CONTAINER=1
ENV NODE_ENV=production

COPY --from=sys /target/etc /etc
COPY --from=sys /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=sys /blank /tmp
COPY --from=sys /blank /app/data

COPY --from=node /usr/local/bin/node /usr/bin/node
COPY --from=node /etc/ssl/openssl.cnf /etc/ssl/openssl.cnf
COPY --from=node /lib/ld-musl-x86_64.so.1 /lib/ld-musl-x86_64.so.1
COPY --from=node /usr/lib/libgcc_s.so.1 /usr/lib/libgcc_s.so.1
COPY --from=node /usr/lib/libstdc++.so.6 /usr/lib/libstdc++.so.6

COPY --from=build /app /app

EXPOSE 3001

CMD ["/usr/bin/node", "/app/start.js"]
