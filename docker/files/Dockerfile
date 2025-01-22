FROM ubuntu

# Пакеты
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    libpcre3 \
    libpcre3-dev \
    zlib1g \
    zlib1g-dev \
    libssl-dev \
    wget \
    curl \
    git \
    unzip \
    libreadline-dev

# Переменные окружения как в инструкции https://github.com/openresty/lua-nginx-module?tab=readme-ov-file#installation
# третью строку нагуглил, без нее валимся в ошибку

ENV LUAJIT_LIB=/usr/local/lib/libluajit-5.1.so \
    LUAJIT_INC=/usr/local/include/luajit-2.1 \
    LD_LIBRARY_PATH=/usr/local/lib/:/opt/drizzle/lib/:$LD_LIBRARY_PATH

# Сборка и установка LuaJIT
RUN git clone https://github.com/openresty/luajit2.git /usr/local/src/luajit && \
    cd /usr/local/src/luajit && \
    make && \
    make install

# Модули nginx для установки lua
RUN git clone https://github.com/vision5/ngx_devel_kit.git /usr/local/src/ngx_devel_kit
RUN git clone https://github.com/openresty/lua-nginx-module.git /usr/local/src/lua-nginx-module

# Сборка и установка Nginx
RUN cd /usr/local/src && \
    wget https://nginx.org/download/nginx-1.25.1.tar.gz && \
    tar -zxvf nginx-1.25.1.tar.gz && \
    cd nginx-1.25.1 && \
    ./configure --prefix=/etc/nginx \
                --sbin-path=/usr/sbin/nginx \
                --conf-path=/etc/nginx/nginx.conf \
                --error-log-path=/var/log/nginx/error.log \
                --http-log-path=/var/log/nginx/access.log \
                --pid-path=/var/run/nginx.pid \
                --lock-path=/var/run/nginx.lock \
                --with-http_ssl_module \
                --with-pcre \
                --with-http_realip_module \
                --with-http_stub_status_module \
                --with-http_gzip_static_module \
                --with-http_v2_module \
                --with-ld-opt="-Wl,-rpath,/usr/local/lib/libluajit-5.1.so" \
                --add-module=/usr/local/src/ngx_devel_kit \
                --add-module=/usr/local/src/lua-nginx-module && \
    make && \
    make install

# Установка lua-resty-core # !!! added LUA_LIB_DIR=/usr/local/share/lua/5.1 to make install
RUN git clone https://github.com/openresty/lua-resty-core.git /usr/local/src/lua-resty-core && \
    cd /usr/local/src/lua-resty-core && \
    make && \
    make install LUA_LIB_DIR=/usr/local/share/lua/5.1

# Установка lua-resty-lrucache # !!! added LUA_LIB_DIR=/usr/local/share/lua/5.1 to make install
RUN git clone https://github.com/openresty/lua-resty-lrucache.git /usr/local/src/lua-resty-lrucache && \
    cd /usr/local/src/lua-resty-lrucache && \
    make && \
    make install LUA_LIB_DIR=/usr/local/share/lua/5.1

# Создание пользователя nginx
RUN useradd -r nginx

# Копирование конфигурационного файла Nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Запуск Nginx на переднем плане
CMD ["nginx", "-g", "daemon off;"]

# Открытие порта 81
EXPOSE 81