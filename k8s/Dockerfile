FROM yanxiuzi/debian-build-env

#COPY ./* /WebServer/
ADD WebServer /WebServer

RUN sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
        && apt-get -y update \
        && apt-get -y upgrade \
        && apt-get -y install universal-ctags \
        && apt-get -y install libmariadb-dev-compat \
        && cd /WebServer \
        && make
        
CMD cd /WebServer && ./bin/server
