FROM imriss/archlinux

RUN echo 'Scylla on Arch Linux'

MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="Scylla / Arch Linux"

RUN  pacman -Syyu --noconfirm \
  && pacman -S findutils nano vi --needed --noconfirm \
  && pacman-db-upgrade \
  && export editor=nano \
  && pacman -S --needed --noconfirm systemd python python-yaml wget 
  
# DDADD https://raw.githubusercontent.com/imriss/ZoneMinder/master/aur.sh /usr/sbin/aur.sh
# DDADD https://raw.githubusercontent.com/imriss/ZoneMinder/master/add-aur.sh /usr/sbin/add-aur
ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN add-aur docker

RUN cd /home \
  && chown docker docker \
  && cd docker/sandground \
  && su docker -c 'mkdir systemtap_tmp' \ 
  && cd systemtap_tmp \
  && su docker -c 'wget https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=systemtap-git -O PKGBUILD' \
  && su docker -c 'sed -i -e 's=git://sourceware.org/git/systemtap.git=https://gitlab.com/fche/systemtap.git=g' PKGBUILD' \
  && su docker -c 'wget https://aur.archlinux.org/cgit/aur.git/plain/systemtap.install?h=systemtap-git -O systemtap.install' \
  && pacman -S --needed --noconfirm libdaemon nspr avahi elfutils nss \
  && su docker -c 'makepkg -si' 

RUN sudo pacman -U /home/docker/sandground/systemtap_tmp/systemtap-git-2.7.1180.g4e0d00272-1-x86_64.pkg.tar --noconfirm \
  && su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm jre8-openjdk-headless jdk8-openjdk' \
  && archlinux-java set java-8-openjdk/jre 

# 'http://ala.seblu.net/packages/t/thrift/thrift-0.9.1-3-x86_64.pkg.tar.xz' 
RUN su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm supervisor dpkg \
  boost ninja libantlr3c libyaml lz4 zlib \
  snappy jsoncpp gnutls ninja ragel libaio crypto++ xfsprogs numactl hwloc libpciaccess libxml2 \
  python-pyparsing lksctp-tools protobuf libunwind python-yaml yaml-cpp thrift cmake antlr3' \
  && cd /home/docker/sandground \
  && pacman -Rdd --noconfirm thrift \
  && wget -c 'https://archive.archlinux.org/packages/t/thrift/thrift-0.9.1-3-x86_64.pkg.tar.xz' \ 
  && pacman -U --noconfirm thrift-0.9.1-3-x86_64.pkg.tar.xz \
  && cd /home/docker/sandground \
  && git clone --depth=1 --branch=master https://github.com/antlr/antlr3 \
  && cp antlr3/runtime/Cpp/include/* /usr/include/ 

RUN cd /home/docker/sandground \
  && git clone --depth=50 --branch=master https://github.com/scylladb/scylla \
  && cd scylla \
  && git submodule sync --recursive \
  && git submodule update --init --recursive 
  
RUN cd /home/docker/sandground/scylla/seastar/fmt \
  && wget https://github.com/fmtlib/fmt/commit/abbefd71666055daac9e14e78262620f9e845850.patch \
  && git config user.name "Reza Farrahi" \
  && git config user.email "imriss@ieee.org" \
  && git am < abbefd71666055daac9e14e78262620f9e845850.patch

# RUN echo "`find / -name 'antlr3.hpp' 2>/dev/null`" \
#  && echo "`ls -la /usr/include/*.hpp`"

RUN cd /home/docker/sandground/scylla \
  && ./configure.py --mode=release --with=scylla --disable-xen --static-stdc++ --static-boost --static --cflags " -fvisibility=default" \
  && ninja build/release/scylla -j1
  
ADD ./scylla_bashrc /scylla_bashrc
RUN cat scylla_bashrc >> /etc/bashrc

# Scylla configuration:
ADD ./etc/sysconfig/scylla-server /etc/sysconfig/scylla-server

# Supervisord configuration:
ADD ./etc/supervisord.conf /etc/supervisord.conf
RUN mkdir -p /etc/supervisor.conf.d
ADD ./etc/supervisord.conf.d/scylla-server.conf /etc/supervisord.conf.d/scylla-server.conf
ADD ./etc/supervisord.conf.d/scylla-jmx.conf /etc/supervisord.conf.d/scylla-jmx.conf
RUN mkdir -p /var/log/scylla
ADD ./scylla-service.sh /scylla-service.sh
ADD ./scylla-jmx-service.sh /scylla-jmx-service.sh

RUN useradd -m scylla \
  && echo "scylla:" | chpasswd -e \
  # && /bin/bash -c "comm -23 <(pacman -Qq) <(cat <(pactree -u scylla |sort) <(pactree -u mpfr |sort) <(pactree -u pacman |sort) <(pactree -u sudo |sort) <(pacman -Qgq base | sort) |sort -u) | xargs pacman -Rdd --noconfirm" \
  && pacman -Scc --noconfirm \
  &&  rm /var/cache/pacman/pkg/* \
  && paccache -rf \
  && rm -rd /home/docker/sandground \
  && echo `du /usr/lib -hd 1`

ADD ./scyllasetup.py /scyllasetup.py
ADD ./commandlineparser.py /commandlineparser.py
ADD ./docker-entrypoint.py /docker-entrypoint.py

