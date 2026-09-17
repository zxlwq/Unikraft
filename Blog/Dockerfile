FROM --platform=linux/x86_64 zxlwq/blog:latest AS content

FROM --platform=linux/x86_64 nginx:alpine

LABEL maintainer="blog"

RUN ln -sfn /proc/self/fd/1 /dev/stdout \
 && ln -sfn /proc/self/fd/2 /dev/stderr

COPY --from=content /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY --from=content /usr/share/html /usr/share/html

EXPOSE 3000

CMD ["nginx", "-g", "daemon off; master_process off;"]
