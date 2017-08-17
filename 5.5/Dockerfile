FROM debian:stretch

MAINTAINER keopx <keopx@keopx.net>

ENV DEBIAN_FRONTEND=noninteractive

# Set repositories
RUN \
   echo "deb http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" > /etc/apt/sources.list && \
   echo "deb-src http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" >> /etc/apt/sources.list && \
   echo "deb http://security.debian.org/ stretch/updates main contrib non-free\n" >> /etc/apt/sources.list && \
   echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
   # Update repositories cache and distribution
   apt-get -qq update && apt-get -qqy upgrade

RUN \
   apt-get -yqq install apt-transport-https lsb-release ca-certificates python3-software-properties gnupg2 && \
   echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list && \
   echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list && \
   apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 && \
   apt-get -yqq update

RUN \
   (echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections) && \
   apt-get -yqq install --no-install-recommends -y oracle-java8-installer && \
   rm -rf /var/cache/oracle-jdk8-installer && \
   echo "networkaddress.cache.ttl=60" >> /usr/lib/jvm/java-8-oracle/jre/lib/security/java.security   

# JAVA ENV.
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
# 'me' User in the future.
#RUN groupadd -g 1000 elasticsearch && useradd elasticsearch -u 1000 -g 1000

# elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html
RUN \
   wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add - && \
   echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-5.x.list && \
   apt-get -yqq update && \
   apt-get -yqq install -y --no-install-recommends elasticsearch

# Clear.
RUN \
   apt-get clean && rm -rf /var/lib/apt/lists/*

# Run elasticsearch: https://github.com/docker-library/elasticsearch/blob/4369e593bab8b19ea5dc06d70cc2039c3460af46/5/Dockerfile
ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

RUN set -ex \
	&& for path in \
		./data \
		./logs \
		./config \
		./config/scripts \
	; do \
		mkdir -p "$path"; \
		chown -R elasticsearch:elasticsearch "$path"; \
	done

COPY config ./config

VOLUME /usr/share/elasticsearch/data

COPY scripts/docker-entrypoint.sh /

## Custom elasticsearch user.
USER elasticsearch

EXPOSE 9200 9300
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["elasticsearch"]
