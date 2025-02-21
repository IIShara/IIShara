# Сборка RPM

## Процесс пересборки RPM пакетов
- Загрузим src.rpm файл с исходниками
- Установим зависимости
- Соберем пакет
- Установим и проверим работу сервиса

## Сборка RPM-пакета с нуля
- Создаем директорию для сборки
- Получаем исходный код
- Пишем spec-файл
- Собираем пакет
- Устанавливаем пакет
---

## Пересборка RPM пакета NGINX
### Перед началом работ необходимо установить:
```bash
yum install yum-utils wget tar git cmake gcc rpm-build perl
```

### Скачать source пакет Nginx


**Скачать source пакет:**

```bash
yumdownloader --source nginx
```

В результате получим:
```
# ll
total 1088
-rw-r--r--. 1 root root 1112203 Feb 20 07:33 nginx-1.20.1-20.el9.src.rpm

```
**Распаковка source rpm:**
```bash
rpm -Uvh nginx*.src.rpm
```
`-U` - upgrade package(s)

`-v` - provide more detailed output

`-h` - print hash marks as package installs (good with -v)

В результате получим:
```
# cd ~
# ll rpmbuild/
total 4
drwxr-xr-x. 2 root root 4096 Feb 20 07:36 SOURCES
drwxr-xr-x. 2 root root   24 Feb 20 07:36 SPECS
```

### Доюавление openssl последней версии
**Скачать openssl последней версии, в этом случае openssl 3.4.1:**
```
wget https://github.com/openssl/openssl/releases/download/openssl-3.4.1/openssl-3.4.1.tar.gz
```

**Распаковка openssl:**
```
tar -xzvf openssl-3.4.1.tar.gz
```
`-x` - extract files from an archive

`-z` - filter the archive through gzip

`-v` - verbosely list files processed

`-f` - use archive file or device ARCHIVE


### Добавление ngx_brotli
**Скачать ngx_brotli:**
```
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
```
`--recurse-submodules` - After the clone is created, initialize and clone submodules within based
on the provided <pathspec>. If no =<pathspec> is provided, all
submodules are initialized and cloned. This option can be given multiple
times for pathspecs consisting of multiple entries. The resulting clone
has submodule.active set to the provided pathspec, or "." (meaning all
submodules) if no pathspec is provided.

Submodules are initialized and cloned using their default settings. This
is equivalent to running git submodule update --init --recursive
<pathspec> immediately after the clone is finished. This option is
ignored if the cloned repository does not have a worktree/checkout (i.e.
if any of --no-checkout/-n, --bare, or --mirror is given)

`-j8` - The number of submodules fetched at the same time. Defaults to the submodule.fetchJobs option

### Сборка ngx_brotli
Команды взяты из [GitHub](https://github.com/google/ngx_brotli?tab=readme-ov-file#installation)

```bash
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
cmake --build . --config Release --target brotlienc
cd ../../../.
```

### Установка зависимостей для сборки nginx
```bash
yum-builddep nginx
```

## Сборка nginx
**Редактируем `./rpmbuild/SPECS/nginx.spec`**
```bash
vi ./rpmbuild/SPECS/nginx.spec
```
**В секцию ./configure добавляем следующее:**

`--add-module=/root/ngx_brotli \` - указываем путь до ngx_brotli

`--with-openssl=/root/openssl-3.4.1 \` - Указываем путь до openssl

`--with-openssl-opt='-fno-lto' \` - отключаем предупреждения openssl

```bash
if ! ./configure \
    --prefix=%{_datadir}/nginx \
    --sbin-path=%{_sbindir}/nginx \
    --modules-path=%{nginx_moduledir} \
    --conf-path=%{_sysconfdir}/nginx/nginx.conf \
    --error-log-path=%{_localstatedir}/log/nginx/error.log \
    --http-log-path=%{_localstatedir}/log/nginx/access.log \
    --http-client-body-temp-path=%{_localstatedir}/lib/nginx/tmp/client_body \
    --http-proxy-temp-path=%{_localstatedir}/lib/nginx/tmp/proxy \
    --http-fastcgi-temp-path=%{_localstatedir}/lib/nginx/tmp/fastcgi \
    --http-uwsgi-temp-path=%{_localstatedir}/lib/nginx/tmp/uwsgi \
    --http-scgi-temp-path=%{_localstatedir}/lib/nginx/tmp/scgi \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/subsys/nginx \
    --user=%{nginx_user} \
    --group=%{nginx_user} \
    --with-compat \
    --add-module=/root/ngx_brotli \
    --with-openssl=/root/openssl-3.4.1 \
    --with-openssl-opt='-fno-lto' \
    --with-debug \
```

**Собираем rpm пакет:**
```
cd ./rpmbuild/SPECS/
rpmbuild -ba nginx.spec -D 'debug_package %{nil}'
```

Результат:
```bash
[root@localhost rpmbuild]# ll
total 4
drwxr-xr-x. 3 root root   43 Feb 20 08:46 BUILD
drwxr-xr-x. 2 root root    6 Feb 20 08:54 BUILDROOT
drwxr-xr-x. 4 root root   34 Feb 20 08:54 RPMS
drwxr-xr-x. 2 root root 4096 Feb 20 07:36 SOURCES
drwxr-xr-x. 2 root root   24 Feb 20 08:37 SPECS
drwxr-xr-x. 2 root root   41 Feb 20 08:53 SRPMS

```

```
# ll RPMS/
total 4
drwxr-xr-x. 2 root root  105 Feb 20 08:54 noarch
drwxr-xr-x. 2 root root 4096 Feb 20 08:54 x86_64
```

```
[root@localhost rpmbuild]# cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/
[root@localhost rpmbuild]# cd RPMS/x86_64/
[root@localhost x86_64]# ll
total 4136
-rw-r--r--. 1 root root   36141 Feb 20 08:54 nginx-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root    7209 Feb 20 22:57 nginx-all-modules-1.20.1-20.el9.noarch.rpm
-rw-r--r--. 1 root root 3200520 Feb 20 08:54 nginx-core-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root    8324 Feb 20 22:57 nginx-filesystem-1.20.1-20.el9.noarch.rpm
-rw-r--r--. 1 root root  759182 Feb 20 08:54 nginx-mod-devel-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root   19233 Feb 20 08:54 nginx-mod-http-image-filter-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root   30891 Feb 20 08:54 nginx-mod-http-perl-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root   18043 Feb 20 08:54 nginx-mod-http-xslt-filter-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root   53641 Feb 20 08:54 nginx-mod-mail-1.20.1-20.el9.x86_64.rpm
-rw-r--r--. 1 root root   80195 Feb 20 08:54 nginx-mod-stream-1.20.1-20.el9.x86_64.rpm
```

## Установка и проверка

```
yum localinstall *.rpm
```

```bash
# nginx -V
nginx version: nginx/1.20.1
built by gcc 11.5.0 20240719 (Red Hat 11.5.0-5) (GCC)
built with OpenSSL 3.4.1 11 Feb 2025 #Версия OpenSSL, которую мы добавили
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-p                                                                                                                        ath=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/l                                                                                                                        og/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-p                                                                                                                        ath=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --h                                                                                                                        ttp-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx                                                                                                                        /tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --l                                                                                                                        ock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-compat --add-module=                                                                                                                        /root/ngx_brotli --with-openssl=/root/openssl-3.4.1 --with-openssl-opt=-fno-lto --with                                                                                                                        -debug --with-file-aio --with-http_addition_module --with-http_auth_request_module --w                                                                                                                        ith-http_dav_module --with-http_degradation_module --with-http_flv_module --with-http_                                                                                                                        gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic -                                                                                                                        -with-http_mp4_module --with-http_perl_module=dynamic --with-http_random_index_module                                                                                                                         --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --wi                                                                                                                        th-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v                                                                                                                        2_module --with-http_xslt_module=dynamic --with-mail=dynamic --with-mail_ssl_module --                                                                                                                        with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-stream                                                                                                                        _ssl_preread_module --with-threads --with-cc-opt='-O2 -flto=auto -ffat-lto-objects -fe                                                                                                                        xceptions -g -grecord-gcc-switches -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_                                                                                                                        SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -fst                                                                                                                        ack-protector-strong -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -march=x86-64-                                                                                                                        v2 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protectio                                                                                                                        n' --with-ld-opt='-Wl,-z,relro -Wl,--as-needed -Wl,-z,now -specs=/usr/lib/rpm/redhat/r                                                                                                                        edhat-hardened-ld -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -Wl,-E'

```