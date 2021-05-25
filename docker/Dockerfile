FROM ubuntu:focal

# register arguments (from make)
ARG REPO

# add powerdns repository
ADD powerdns.pref /etc/apt/preferences.d/powerdns
ADD https://repo.powerdns.com/FD380FBB-pub.asc /etc/apt/trusted.gpg.d/powerdns.asc
RUN echo "deb [arch=amd64] http://repo.powerdns.com/ubuntu ${REPO} main" >> /etc/apt/sources.list \
  && chmod 0644 /etc/apt/trusted.gpg.d/powerdns.asc

# install pdns-server and tools
RUN apt-get update \
  && apt-get install -y --no-install-recommends pdns-server pdns-backend-mysql mysql-client \
  && rm -rf /var/lib/apt/lists/*

# prepare ports
EXPOSE 53/udp 53/tcp

# setup entrypoint
COPY docker-entrypoint.sh /
ENTRYPOINT [ "/docker-entrypoint.sh" ]

# default command
CMD ["/usr/sbin/pdns_server", "--setuid=pdns", "--setgid=pdns", "--daemon=no", "--launch=gmysql"]
