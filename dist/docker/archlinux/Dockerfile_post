FROM imriss/scylla_archlinux:compact

RUN echo 'Scylla on Arch Linux'

MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="Scylla / Arch Linux"

ENTRYPOINT ["/docker-entrypoint.py"]

EXPOSE 10000 9042 9160 9180 7000 7001
VOLUME [ "/var/lib/scylla" ]
RUN chown -R scylla.scylla /var/lib/scylla
