# 编译ln/amp环境

* 代码下载
[apr](https://apr.apache.org/)
[curl](https://curl.haxx.se/download.html)
[jpeg](http://www.ijg.org/)
[uuid](http://pecl.php.net/package/uuid)
[pcre](http://www.pcre.org/)
[libgd](https://libgd.github.io/pages/downloads.html)
[sqlite](http://www.sqlite.org/chronology.html)
[libxpm](https://lists.freedesktop.org/archives/xorg-announce/2012-March/001858.html)
[libpng](http://dl.ambiweb.de/mirrors/www.libpng.org/pub/png/libpng.html)
[libxml2](http://xmlsoft.org/downloads.html)
[openssl](https://www.openssl.org/source/) 此处注意下载1.0.x版本的
[freetype](https://www.freetype.org/download.html)
[libiconv](http://www.gnu.org/software/libiconv/#downloading)

* 编译代码

```shell
#! /bin/bash

HTTPDPATH="/home/xyg/httpd"
BASEPATH="/home/xyg/x86"
LOGPATH="$BASEPATH/build.log"

test -d $BASEPATH || mkdir -p $BASEPATH
test -d $HTTPDPATH || mkdir -p $HTTPDPATH

chmod -R 700 $BASEPATH
chown -R root:root $BASEPATH

function message
{
    echo ""
    echo "********************************"
    echo "********************************"
    echo "************* $* ***************"
    echo "********************************"
    echo "********************************"
    
    if [ -f $LOGPATH ]; then
        rm -rf $LOGPATH
    fi

    echo "$(date +"[ %F %T %:z ]") $*" | tee -a $LOGPATH
    
    sleep 1
}

function error
{
    echo "" >&2
    echo "BackTrace" | tee -a $LOGPATH
    
    local i
    for (( i=0; i<${#FUNCNAME[@]}; i++ )); do
        printf "In File $0, On line %s \n , ${BASH_LINENO[$i]} ${FUNCNAME[$i]}" | tee -a $LOGPATH
        
    if [ $i eq 0 ]; then
        echo $*
    fi
}

function compile-httpd-deps
{
    compile-openssl
    compile-pcre
    compile-apr
    compile-apr-util
    compile-zlib
}

function compile-openssl
{
    message "Compile Openssl"
    
    cd $BASEPATH/code/openssl || error "Can not find openssl file"
    
    chmod +x util/point.sh
    
    perl config -fPIC --prefix=$BASEPATH/compile-openssl enable-shared no-ssl2
    #-fPIC 编译静态库 enable-shared 编译动态库
    make clean
    
    make && make install || error 'Compile openssl failed'  
    
    # openssl version 查看openssl目前版本
    # find / -name openssl 查找所有openssl的位置，一般在/usr/bin下
    # mv /usr/bin/openssl /usr/bin/openssl.backup
    # ln -s $BASEPATH/compile-openssl/bin/openssl /usr/bin/openssl
    # openssl version 再次查看openssl版本是否升级成功
    # 
    # 以下两步非必须
    # echo "$BASEPATH/compile-openssl/bin/openssl" >> /etc/ld.so.conf
    # ldconfig -v
}

function compile-pcre
{
    message "Compile PCRE"
    
    cd $BASEPATH/code/pcre || error "Can not find pcre2 file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile pcre2 failed'  
}

function compile-apr
{
    message "Compile APR"
    
    cd $BASEPATH/code/apr || error "Can not find apr file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile apr failed'  
}

function compile-apr-util
{
    message "Compile APR-UTIL"
    
    cd $BASEPATH/code/apr-util || error "Can not find apr-util file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --with-apr=$BASEPATH/libraries
    
    make clean
    
    make && make install || error 'Compile apr-util failed'  
    #xml/apr_xml.c:35:19: 致命错误：expat.h：没有那个文件或目录
    #缺expat的开发库
    #yum install expat-devel
}

function compile-zlib
{
    message "Compile Zlib"
    
    cd $BASEPATH/code/zlib || error "Can not find zlib file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile zlib failed'  
}

function compile-httpd
{
    message "Compile HTTPD"
    
    cd $BASEPATH/code/httpd || error "Can not find httpd file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    export CFLAGS="-lm"
    export LDFLAGS="-lm" 
    # 出现如下问题：
    # libaprutil-1.so: undefined reference to `XML_ParserCreate
    # collect2: error: ld returned 1 exit status
    # 解决方案：
    # export LDFLAGS="-L$BASEPATH/libraries/bin/" 
    
    ./configure --prefix=/home/xyg/httpd --with-pcre=$BASEPATH/libraries --with-apr=$BASEPATH/libraries --with-apr-util=$BASEPATH/libraries --enable-ssl --with-ssl=$BASEPATH/compile-openssl --enable-mpms-shared=all --enable-so --enable-rewrite --with-z=$BASEPATH/libraries --enable-deflate --enable-vhost-alias
    
    make clean
    
    make && make install || error 'Compile httpd failed'  
}

function compile-php-deps
{
    compile-libxml2
	#compile-libmcypt    #php7.2.1已经废弃掉了（移入到pecl里面了），其函数用openssl函数代替
    compile-libpng
    compile-jpeg
    compile-freetype
    compile-libiconv
    compile-libgd
    compile-curl
    compile-libxpm
    #sqlite
}

function compile-libxml2
{
    message "Compile Libxml2"
    
    cd $BASEPATH/code/libxml2 || error "Can not find libxml2 file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared --without-python
    
    make clean
    
    make && make install || error 'Compile libxml2 failed'  
}

function compile-libmcypt
{
    message "Compile libmcypt"
    
    cd $BASEPATH/code/libmcypt || error "Can not find libmcypt file"
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared --disable-posix-threads
    
    make clean
    
    make && make install || error 'Compile libmcypt failed'  
}

#  此处修改了linpng的源码 gcc4.8以上不需要修改
#  libpng-1.6.21\contrib\tools\pngfix.c
#  第2186行 源码是   
#  zlib->rc = inflateReset2(&zlib->z, 0);
#  修改为
#  #if PNG_ZLIB_VERNUM < 0x1240
#      zlib->rc = inflateReset(&zlib->z, 0);
#  #else
#      zlib->rc = inflateReset2(&zlib->z, 0);
#  #endif

#  1.5版本不需要以上修改，直接编译即可
function compile-libpng
{
    message "Compile Libpng" || error "Can not find libpng file"
    
    cd $BASEPATH/code/libpng
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared --with-zlib-prefix=$BASEPATH/libraries
    
    # LDFLAGS="-L$BASEPATH/libraries/lib"
    # CPPFLAGS="-I$BASEPATH/libraries/include"
    
    make clean
    
    make && make install || error 'Compile libpng failed'  
    # error: zlib not installed
    # 解决方法：重新进入zlib，configure时不带prefix参数重新编译安装
}

function compile-jpeg
{
    message "Compile JPEG"
    
    cd $BASEPATH/code/jpeg || error "Can not find jpeg file"
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile jpeg failed'  
}

function compile-freetype
{
    message "Compile Freetype"
    
    cd $BASEPATH/code/freetype || error "Can not find freetype file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --with-zlib=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile freetype failed'  
}

function compile-libiconv
{
    message "Compile Libiconv"
    
    cd $BASEPATH/code/libiconv || error "Can not find libiconv file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile libiconv failed'  
    
    #remember to run 'libtool --finish $BASEPATH/libraries/lib'
}

#  
#  vi src/Makefile
#  找到其中的png12字样，然后删除其相关字段
function compile-libgd
{
    message "Compile Libgd"
    
    cd $BASEPATH/code/libgd || error "Can not find libgd file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --with-png=$BASEPATH/libraries --with-jpeg=$BASEPATH/libraries --with-freetype=$BASEPATH/libraries --with-libiconv-prefix=$BASEPATH/libraries --with-zlib=$BASEPATH/libraries --enable-shared
    
    #sed -i 's/LIBS = \(.*\)/LIBS = \1 -liconv/' src/Makefile
    
    make clean
    
    make
	
	make install || error 'Compile libgd failed'  
}

function compile-curl
{
    message "Compile Curl"
    
    cd $BASEPATH/code/curl || error "Can not find curl file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    #export LDFLAGS=-R$BASEPATH/compile-openssl/lib
    export LDFLAGS="-L$BASEPATH/libraries/lib"
    ./configure --prefix=$BASEPATH/libraries --with-ssl=$BASEPATH/compile-openssl --without-gnutls
    
    make clean
    
    make && make install || error 'Compile curl failed'  
}

function compile-libxpm
{
    message "Compile libXpm"
    
    cd $BASEPATH/code/libXpm || error "Can not find libXpm file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared
    
    make clean
    
    make && make install || error 'Compile libXpm failed'  

    # configure: error: Package requirements (xproto x11) were not met:
    # No package 'xproto' found
    # No package 'x11' found
    # 解决方案：
    # yum -y install libX11 libX11-common libX11-devel
    
    # /bin/sh: xgettext: command not found
    # make[2]: *** [cxpm.po] 错误 127
    # make[2]: Leaving directory `$BASEPATH/code/libXpm-3.5.12/cxpm'
    # make[1]: *** [all-recursive] 错误 1
    # make[1]: Leaving directory `$BASEPATH/code/libXpm-3.5.12'
    # make: *** [all] 错误 2
    # 解决方案：
    # yum -y install gettext.x86_64 gettext-libs.x86_64 gettext-devel.x86_64
}

function compile-uuid
{
#yum -y install libuuid.x86_64 libuuid-devel.x86_64
    message "Compile PHP"
    
    cd $BASEPATH/code/uuid || error "Can not find uuid file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    $BASEPATH/compile-php/bin/phpize

    ./configure --prefix=$BASEPATH/libraries --with-php-config=$BASEPATH/compile-php/bin/php-config
    
    make clean
    
    make && make install || error 'Compile uuid failed'  
}

function compile-phpjwt
{
    cd $BASEPATH/code/php-jwt-master || error "Can not find php-jwt file"
    
    
    $BASEPATH/compile-php/bin/phpize

    ./configure --prefix=$BASEPATH/libraries --with-php-config=$BASEPATH/compile-php/bin/php-config
    
    make clean
    
    make && make install || error 'Compile uuid failed'  
}

function compile-php
{
    message "Compile PHP"
    
    cd $BASEPATH/code/php || error "Can not find php file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    #./configure --prefix=$BASEPATH/compile-php --with-apxs2=/home/xyg/httpd/bin/apxs --with-libxml-dir=$BASEPATH/libraries --with-openssl=$BASEPATH/compile-openssl --with-curl=$BASEPATH/libraries --with-jpeg-dir=$BASEPATH/libraries --with-png-dir=$BASEPATH/libraries --with-gd=$BASEPATH/libraries --with-freetype-dir=$BASEPATH/libraries --with-iconv-dir=$BASEPATH/libraries --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mhash --enable-mbstring --enable-mysqlnd --with-config-file-path=/home/xyg/httpd/conf --enable-sockets --enable-ftp --enable-soap --enable-bcmath --enable-zip --with-zlib=$BASEPATH/libraries --with-zlib-dir=$BASEPATH/libraries --disable-fileinfo --enable-shmop --disable-rpath --with-xpm-dir=$BASEPATH/libraries --disable-cli --disable-cgi
    
    ./configure --prefix=$BASEPATH/compile-php --with-apxs2=/home/xyg/httpd/bin/apxs --with-libxml-dir=$BASEPATH/libraries --with-openssl=$BASEPATH/compile-openssl --with-curl=$BASEPATH/libraries --with-jpeg-dir=$BASEPATH/libraries --with-png-dir=$BASEPATH/libraries --with-gd --with-freetype-dir=$BASEPATH/libraries --with-iconv-dir=$BASEPATH/libraries --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mhash --enable-mbstring --enable-mysqlnd --with-config-file-path=/home/xyg/httpd/conf --enable-sockets --enable-ftp --enable-soap --enable-bcmath --enable-zip --with-zlib=$BASEPATH/libraries --with-zlib-dir=$BASEPATH/libraries --disable-fileinfo --enable-shmop --disable-rpath --with-xpm-dir=$BASEPATH/libraries --disable-cli --disable-cgi
    
	#--with-libdir=lib64 解决Don't know how to define struct flock on this system, set --enable-opcache=no
    #此解决方案会导致下面一系列（lib64）问题，所以必须找其他替代方案
    #真正的解决方案是：
    # echo "$BASEPATH/libraries/lib" >> /etc/ld.so.conf
    # echo "$BASEPATH/compile-openssl/lib" >> /etc/ld.so.conf
    # ldconfig -v
    
    # php error unable to find libgd.(a so) = 2.1.0 anywhere under $BASEPATH/libraries
    # 解决方案：
    # --with-gd=$BASEPATH/libraries  删除后面指定的路径             会引起其他错误
    # rpm -qa|grep libpng   , rpm -e卸载低版本的libpng                  方法错误
    # ln -s $BASEPATH/libraries/lib $BASEPATH/libraries/lib64   方法错误
    
    # error: Cannot find OpenSSL's libraries
    # 解决方案：
    # ln -s $BASEPATH/compile-openssl/lib/ $BASEPATH/compile-openssl/lib64
    
    make clean
    
    # undefined reference to 'libiconv_open'
    # 解决方案：
    # make ZEND_EXTRA_LIBS='-liconv'
    make && make install || error 'Compile php failed'  
    
    #./libtool --finish $BASEPATH/code/php-7.2.7/libs
}

function compile-php-opcache
{
    message "Compile PHP"
    
    cd $BASEPATH/code/php/ext/opcache || error "Can not find php-opcache file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    $BASEPATH/compile-php/bin/phpize

    ./configure --with-php-config=$BASEPATH/compile-php/bin/php-config
    
    make clean
    
    make && make install || error 'Compile php-opcache failed'  
}

function compile-php-openssl
{
    message "Compile PHP"
    
    cd $BASEPATH/code/php/ext/openssl || error "Can not find php-openssl file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    mv config0.m4 config.m4
    
    $BASEPATH/compile-php/bin/phpize

    ./configure --with-openssl=$BASEPATH/compile-openssl --with-php-config=$BASEPATH/compile-php/bin/php-config
    
    make clean
    
    make && make install || error 'Compile php-openssl failed'  
}

# 此包可不安装
function compile-fontconfig
{
    message "Compile fontconfig"
    
    cd $BASEPATH/code/fontconfig || error "Can not find fontconfig file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    export PKG_CONFIG_PATH=$BASEPATH/libraries/lib/pkgconfig/:$PKG_CONFIG_PATH
    
    ./configure --prefix=$BASEPATH/libraries --enable-shared --enable-iconv --enable-libxml2
    
    make clean
    
    make && make install || error 'Compile fontconfig failed'

#  如果缺少itstool 优先安装python2.7.x
#  cd python
#  ./configure
#  make && make install
#  
#  mv /usr/bin/python /usr/bin/python2.6.6
#  ln -s /usr/local/bin/python2.7 /usr/bin/python
#  
#  python -V #查看python版本
#  
#  为了避免yum出差，因为yum只支持python2.6
#  vi /usr/bin/yum
#  #!/usr/bin/python
#  改成
#  #!/usr/bin/python2.6.6
}

## 安装memcache开始
function compile-libevent
{
    message "Compile libevent"
    
    cd $BASEPATH/code/libevent || error "Can not find libevent file"
    
    
    ./configure --prefix=$BASEPATH/libevent 
    
    make clean
    
    make && make install || error 'Compile libevent'  
}

function compile-libmemcached
{
    #https://launchpad.net/libmemcached/+download 下载地址
    
    message "Compile memcached"
    
    cd $BASEPATH/code/libmemcached || error "Can not find libmemcached file"
    
    
    ./configure --prefix=$BASEPATH/libraries
    
    make clean
    
    make && make install || error 'Compile libmemcached'  
}

function compile-memcached
{
    #https://github.com/php-memcached-dev/php-memcached/tree/php7 下载地址
    
    message "Compile memcached"
    
    cd $BASEPATH/code/memcached || error "Can not find memcached file"
    
    
    ./configure --prefix=$BASEPATH/memcache --with-php-config=$BASEPATH/compile-php/bin/php-config --with-libmemcached-dir=$BASEPATH/libraries --disable-memcached-sasl
    
    make clean
    
    make && make install || error 'Compile memcached'  
}

function compile-memcache
{
    #https://github.com/gophp7/gophp7-ext/wiki/extensions-catalog 注意在git上下载支持php7的memcache
    #https://github.com/websupport-sk/pecl-memcache/tree/php7 下载地址
    message "Compile memcache"
    
    cd $BASEPATH/code/memcache || error "Can not find memcache file"
    
    
    $BASEPATH/compile-php/bin/phpize
    
    ./configure --prefix=$BASEPATH/memcache --enable-memcache --with-php-config=$BASEPATH/compile-php/bin/php-config --with-zlib-dir=$BASEPATH/libraries
    
    make clean
    
    make && make install || error 'Compile memcache'  
}

## 安装memcache结束
function compile-nginx
{
	message "Compile nginx"
    
    cd $BASEPATH/code/nginx || error "Can not find nginx file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
	
	./configure --prefix=/home/xyg/nginx \
    --user=nginx \
    --group=nginx \
    --with-http_realip_module \
    --with-http_gzip_static_module \
    --with-pcre=$BASEPATH/code/pcre-8.41 \
    --with-zlib=$BASEPATH/code/zlib-1.2.11 \
    --with-http_ssl_module \
    --with-openssl=$BASEPATH/code/openssl-1.0.2o
	
	make clean
    
    make
	
	make install || error 'Compile nginx failed'  
}

function compile-fcgi-php
{
    message "Compile FastCGI PHP"
    
    cd $BASEPATH/code/php || error "Can not find PHP file"
    
    find . -name "*sh" -o -name "config*" | xargs -r chmod +x
    
    make distclean
	
    ./configure --prefix=/home/xyg/php-fpm 
    --enable-fpm \
    --with-fpm-user=nginx \
    --with-fpm-group=nginx \
    --with-libxml-dir=$BASEPATH/libraries \
    --with-openssl=$BASEPATH/compile-openssl \
    --with-curl=$BASEPATH/libraries \
    --with-gd \
    --with-jpeg-dir=$BASEPATH/libraries \
    --with-png-dir=$BASEPATH/libraries \
    --with-freetype-dir=$BASEPATH/libraries \
    --with-iconv-dir=$BASEPATH/libraries \
    --with-config-file-path=/home/xyg/php-fpm/conf \
    --with-pdo-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --with-mhash \
    --enable-mbstring \
    --enable-zip \
    --enable-mysqlnd \
    --enable-sockets \
    --enable-ftp \
    --enable-soap \
    --enable-bcmath \
    --with-zlib=$BASEPATH/libraries \
    --with-zlib-dir=$BASEPATH/libraries \
    --disable-fileinfo \
    --enable-shmop \
    --disable-rpath \
    --with-xpm-dir=$BASEPATH/libraries \
    --enable-maintainer-zts \
    --enable-pcntl
    
    #--enable-maintainer-zts 解决opcache和redis启动时错误 opcache undefined symbol: core_globals_id
    # Installing shared extensions:     /home/xyg/php-fpm/lib/php/extensions/no-debug-zts-20170718/
    # Installing PHP CLI binary:        /home/xyg/php-fpm/bin/
    # Installing PHP CLI man page:      /home/xyg/php-fpm/php/man/man1/
    # Installing PHP FPM binary:        /home/xyg/php-fpm/sbin/
    # Installing PHP FPM defconfig:     /home/xyg/php-fpm/etc/
    # Installing PHP FPM man page:      /home/xyg/php-fpm/php/man/man8/
    # Installing PHP FPM status page:   /home/xyg/php-fpm/php/php/fpm/
    # Installing phpdbg binary:         /home/xyg/php-fpm/bin/
    # Installing phpdbg man page:       /home/xyg/php-fpm/php/man/man1/
    # Installing PHP CGI binary:        /home/xyg/php-fpm/bin/
    # Installing PHP CGI man page:      /home/xyg/php-fpm/php/man/man1/
    # Installing build environment:     /home/xyg/php-fpm/lib/php/build/
    # Installing header files:          /home/xyg/php-fpm/include/php/
    # Installing helper programs:       /home/xyg/php-fpm/bin/
    
    # make ZEND_EXTRA_LIBS='-liconv'
    make
    make install || error 'Compile FastCGI PHP failed'  
}

function compile-swoole
{
    message "Compile swoole"
    
    cd $BASEPATH/code/swoole/ || error "Can not find swoole file"
    
    $BASEPATH/compile-php/bin/phpize

    ./configure --with-php-config=$BASEPATH/compile-php/bin/php-config \
    --with-openssl-dir=$BASEPATH/compile-openssl \
    --enable-debug-log \
    --enable-sockets \
    --enable-openssl \
    --enable-mysqlnd
    
    make clean
    
    make && make install || error 'Compile swoole failed'  
}


function compile-all
{
    compile-httpd-deps
    compile-httpd
    
    sleep 20
    
    compile-php-deps
    compile-php
    compile-php-openssl
}

function tarball
{
    message "tarball"
    
    cd /home/xyg/
    
    cp -rf httpd httpd_bak
    
    cd httpd
    
    rm -rf build cgi-bin include man manual
    
    cp -p $BASEPATH/libraries/lib/*so* modules
    cp -p $BASEPATH/compile-openssl/lib/*so* modules
    cp -p $BASEPATH/php.ini conf
    
    mkdir httpd/extensions
    
    cd httpd/extensions
    
    cp -p $BASEPATH/compile-php/lib/php/extensions/no-debug-zts-20170718/*so* .
    
    
    cd /home/xyg/php-fpm/lib/php/extensions
    
    cp no-debug-zts-20170718/* .
    cp -p $BASEPATH/compile-php/lib/php/extensions/no-debug-zts-20170718/*so* .
    
    cd /home/xyg/php-fpm/
    mkdir -p conf/php-fpm.d
    cd conf
    cp ../etc/pear.conf .
    cp ../etc/php-fpm.conf.default php-fpm.conf
    cp ../etc/php-fpm.d/www.conf.default php-fpm.d/www.conf
    
    tar -zcf httpd-php.tar.gz $HTTPDPATH
}

function usage
{
    echo "Usage: ./$0 [OPTION]"
    echo ""
    echo "General setting:"
    echo "       " "no option, will be complete compile"
    echo "httpd  " "will be compile httpd-depends and httpd"
    echo "php    " "will be compile php-depends and php"
    echo "tarball" "only tarball"
    echo ""
    echo "./$0"
    echo "./$0 httpd"
    echo "./$0 php"
    echo "./$0 tarball"
}

case $# in
    0)
        compile-all
        tarball
    ;;
    1)
        case $* in
            httpd)
                compile-httpd-deps
                compile-httpd
            ;;
            php)
                compile-php-deps
                compile-php
            ;;
            tarball)
                tarball
            ;;
            *)
                usage
            ;;
    *)
        usage
    ;;
esac
    
##升级GCC到4.8以上版本
##http://ftp.gnu.org/gnu/gcc/gcc-6.2.0/gcc-6.2.0.tar.bz2

#安装gcc依赖的安装环境
yum -y install gcc.x86_64 libgcc.x86_64 gcc-c++.x86_64

cd /home/xyg/gcc
wget http://ftp.gnu.org/gnu/gcc/gcc-6.2.0/gcc-6.2.0.tar.bz2
tar -xjf gcc-6.2.0.tar.bz2
cd gcc-6.2.0
./contrib/download_prerequisites  ##下载gcc依赖包

mkdir build && cd build
./configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j4      #编译 4表示核心数 一般是物理核心数的2倍为宜，查看物理核心数：cat /proc/cpuinfo | grep "core id" | wc -l
make install  #安装

ls /usr/local/bin | grep gcc  #查看安装是否成功
locate gcc-6.2|tail #确定gcc-6.2安装位置 安装 locate yum -y install mlocate updatedb
update-alternatives --install /usr/bin/gcc gcc /usr/local/bin/x86_64-pc-linux-gnu-gcc 40 #添加新GCC到可选项，倒数第三个是名字，倒数第二个参数为新GCC路径，最后一个参数40为优先级，设大一些之后就自动使用新版了
#断开ssh，重启之后gcc -v查看gcc版本

##
#  编译GCC时出现 
#  no acceptable C compiler found in $PATH
#  解决方法：  yum -y install gcc
#  
#  出现
#  make[2]: Leaving directory `/home/xyg/gcc/gcc-6.2.0'
#  make[1]: *** [stage1-bubble] 错误 2
#  make[1]: Leaving directory `/home/xyg/gcc/gcc-6.2.0'
#  make: *** [all] 错误 2
#  解决方法：  yum -y install gcc-c++
#


#安装sqlite,svn需要
cd $BASEPATH/code/sqlite-autoconf-3140200
./configure --prefix=$BASEPATH/libraries
make && make install


#安装subversion
cd $BASEPATH/code/subversion-1.9.4
./configure --prefix=/usr/local/svn1.9.4 --with-apr=$BASEPATH/libraries --with-apr-util=$BASEPATH/libraries --with-sqlite=$BASEPATH/libraries --with-zlib=$BASEPATH/libraries/zlib

echo 'export PATH=$PATH:/usr/local/svn1.9.4' >> /etc/profile
. /etc/profile


#事务测试出错 一般是安装了2个以上不同版本的软件，冲突了，卸载之前版本即可

function compile-redis(){ 
    cd /home/xyg/redis
    
    make 
    make install
    
    # 设置redis开机启动
    #
    # mkdir /etc/redis
    #
    # cp redis.conf /etc/redis/6379.conf
    # cp utils/redis_init_script /etc/init.d/redisd
    #
    # vi /etc/init.d/redisd 
    # !bin/sh 后面添加 # chkconfig: 2345 90 10 保存       
    # chkconfig 后面参数简介： 2345表示系统运行级别是2，3，4或者5时都启动此服务，90，是启动的优先级，10是关闭的优先级，如果启动优先级配置的数太小时如0时，则有可能启动不成功，因为此时可能其依赖的网络服务还没有启动，从而导致自启动失败
    # 
    # 修改daemonize为yes，即默认以后台程序方式运行
    #
    # chkconfig redisd on
    #
    # 之后就可以通过service 命令启动
    # service redisd start/stop
    
}

#phpize 时如果没有生成configure文件，则说明autoconf没安装
#Cannot find autoconf. Please check your autoconf installation and the
#$PHP_AUTOCONF environment variable. Then, rerun this script.
#解决方案： yum -y install autoconf


# ab测试时，apache报错如下apr_socket_recv: Connection reset by peer (104)， 此时在ab后加参数-r
# 这是因为这个参数的意思是当出现“receive error”，即接收数据错误时是否退出，默认是退出的，所以会出现上述的问题
#./ab -n10000 -c 100 http://192.168.1.224:8086/index.php
#This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
#Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
#Licensed to The Apache Software Foundation, http://www.apache.org/

#Benchmarking 192.168.1.224 (be patient)
#apr_socket_recv: Connection reset by peer (104)
#Total of 18 requests completed
```