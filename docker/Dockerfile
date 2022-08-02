FROM alpine:3.16 as builder

ARG RESTY_VERSION="1.21.4.1"
ARG PATCH_VERSION="102101"


RUN set -ex && \
    apk add --no-cache --virtual .build-deps \
        build-base \
        curl \
        gd-dev \
        geoip-dev \
        libxslt-dev \
        linux-headers \
        make \
        perl-dev \
        readline-dev \
        zlib-dev \
        unzip \
        outils-md5 \
        git \
        pcre \
        pcre-dev \
        openssl-dev \
    && apk add --no-cache \
        gd \
        geoip \
        libgcc \
        libxslt \
        zlib

WORKDIR /tmp
RUN git clone --depth=1 https://github.com/chobits/ngx_http_proxy_connect_module.git ngx_http_proxy_connect_module \
  && curl -fSL  https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz -o openresty-${RESTY_VERSION}.tar.gz \
  && tar xf openresty-${RESTY_VERSION}.tar.gz \
  && cd openresty-${RESTY_VERSION}/bundle/nginx-* \
  && patch -p1 < ../../../ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_${PATCH_VERSION}.patch \
  && cd ../.. \
  && ./configure --prefix=/opt/openresty \
    --without-http_ssi_module \
    --without-http_userid_module \
    --without-mail_pop3_module \
    --without-mail_imap_module \
    --without-mail_smtp_module \
    --with-http_sub_module \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_gzip_static_module \
    --with-http_gunzip_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-select_module \
    --with-poll_module \
    --with-file-aio \
    --with-pcre-jit \
    --add-module=../ngx_http_proxy_connect_module \
  && make install \
  && ln -sf /dev/stdout /opt/openresty/nginx/logs/access.log \
  && ln -sf /dev/stderr /opt/openresty/nginx/logs/error.log


FROM alpine:3.16

RUN adduser --disabled-password --ingroup nogroup --no-create-home --uid 64444 -h /dev/null -s /sbin/nologin openresty.proxy \
  && apk add --no-cache libgcc pcre pcre-dev

COPY --from=builder /opt/openresty /opt/openresty

STOPSIGNAL SIGQUIT

ENV PATH="$PATH:/opt/openresty/luajit/bin:/opt/openresty/nginx/sbin:/opt/openresty/bin"

USER openresty.proxy
CMD ["/opt/openresty/bin/openresty", "-g", "daemon off;"]
