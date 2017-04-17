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
  && ls -la \
  && cd docker \
  && ls -la \
  && su docker -c 'mkdir systemtap_tmp' \ 
  && cd systemtap_tmp \
  && su docker -c 'wget https://aur.archlinux.org/cgit/aur.git/plain/PKGBUILD?h=systemtap-git -O PKGBUILD' \
  && su docker -c 'sed -i -e 's=git://sourceware.org/git/systemtap.git=https://gitlab.com/fche/systemtap.git=g' PKGBUILD' \
  && su docker -c 'wget https://aur.archlinux.org/cgit/aur.git/plain/systemtap.install?h=systemtap-git -O systemtap.install' \
  && pacman -S --needed --noconfirm libdaemon nspr avahi elfutils nss \
  && su docker -c 'makepkg -si' \
  && ls -la
RUN sudo pacman -U /home/docker/systemtap_tmp/systemtap-git-2.7.1180.g4e0d00272-1-x86_64.pkg.tar --noconfirm

RUN su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm scylla supervisor'

ADD scylla_bashrc /scylla_bashrc
RUN cat /scylla_bashrc >> /etc/bashrc

# Scylla configuration:
ADD etc/sysconfig/scylla-server /etc/sysconfig/scylla-server

# Supervisord configuration:
ADD etc/supervisord.conf /etc/supervisord.conf
RUN mkdir -p /etc/supervisor.conf.d
ADD etc/supervisord.conf.d/scylla-server.conf /etc/supervisord.conf.d/scylla-server.conf
ADD etc/supervisord.conf.d/scylla-jmx.conf /etc/supervisord.conf.d/scylla-jmx.conf
RUN mkdir -p /var/log/scylla
ADD scylla-service.sh /scylla-service.sh
ADD scylla-jmx-service.sh /scylla-jmx-service.sh

ADD scyllasetup.py /scyllasetup.py
ADD commandlineparser.py /commandlineparser.py
ADD docker-entrypoint.py /docker-entrypoint.py
ENTRYPOINT ["/docker-entrypoint.py"]

EXPOSE 10000 9042 9160 9180 7000 7001
VOLUME [ "/var/lib/scylla" ]
RUN chown -R scylla.scylla /var/lib/scylla

