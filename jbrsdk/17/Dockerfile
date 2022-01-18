FROM debian:buster-slim

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        ca-certificates p11-kit \
    ; \
    rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME /usr/local/jbrsdk-17
ENV PATH $JAVA_HOME/bin:$PATH

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8

ENV JAVA_VERSION 17.0.1

RUN set -eux; \
    \
    downloadUrl='https://d2xrhe97vsfxuc.cloudfront.net/jbrsdk_dcevm-17_0_1-linux-x64-b164.8.tar.gz'; \
    downloadSha256='18b32d16d2bcbec7d0ade644e061831090a26403e3309c27154b1d6a5eca9d65'; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        wget \
    ; \
    rm -rf /var/lib/apt/lists/*; \
    \
    wget --progress=dot:giga -O jbrsdk.tgz "$downloadUrl"; \
    echo "$downloadSha256 *jbrsdk.tgz" | sha256sum --strict --check -; \
    \
    mkdir -p "$JAVA_HOME"; \
    tar --extract \
        --file jbrsdk.tgz \
        --directory "$JAVA_HOME" \
        --strip-components 1 \
        --no-same-owner \
    ; \
    rm jbrsdk.tgz*; \
    \
    downloadUrl='https://github.com/HotswapProjects/HotswapAgent/releases/download/1.4.2-SNAPSHOT/hotswap-agent-1.4.2-SNAPSHOT.jar'; \
	downloadSha256='e4161650308194bc3c8174b521e84cb2f3434f3490404ceaad71db22296f5d8a'; \
	\
	wget --progress=dot:giga -O hotswap-agent.jar "$downloadUrl"; \
	echo "$downloadSha256 *hotswap-agent.jar" | sha256sum --strict --check -; \
	\
	mkdir -p "$JAVA_HOME/lib/hotswap"; \
	mv hotswap-agent.jar "$JAVA_HOME/lib/hotswap/hotswap-agent.jar"; \
    \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    \
# update "cacerts" bundle to use Debian's CA certificates (and make sure it stays up-to-date with changes to Debian's store)
# see https://github.com/docker-library/openjdk/issues/327
#     http://rabexc.org/posts/certificates-not-working-java#comment-4099504075
#     https://salsa.debian.org/java-team/ca-certificates-java/blob/3e51a84e9104823319abeb31f880580e46f45a98/debian/jks-keystore.hook.in
#     https://git.alpinelinux.org/aports/tree/community/java-cacerts/APKBUILD?id=761af65f38b4570093461e6546dcf6b179d2b624#n29
    { \
        echo '#!/usr/bin/env bash'; \
        echo 'set -Eeuo pipefail'; \
        echo 'trust extract --overwrite --format=java-cacerts --filter=ca-anchors --purpose=server-auth "$JAVA_HOME/lib/security/cacerts"'; \
    } > /etc/ca-certificates/update.d/docker-jbrsdk; \
    chmod +x /etc/ca-certificates/update.d/docker-jbrsdk; \
    /etc/ca-certificates/update.d/docker-jbrsdk; \
    \
# https://github.com/docker-library/openjdk/issues/331#issuecomment-498834472
    find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-jbrsdk.conf; \
    ldconfig; \
    \
# https://github.com/docker-library/openjdk/issues/212#issuecomment-420979840
# https://openjdk.java.net/jeps/341
    java -Xshare:dump; \
    \
# basic smoke test
    fileEncoding="$(echo 'System.out.println(System.getProperty("file.encoding"))' | jshell -s -)"; [ "$fileEncoding" = 'UTF-8' ]; rm -rf ~/.java; \
    javac --version; \
    java --version

# "jshell" is an interactive REPL for Java (see https://en.wikipedia.org/wiki/JShell)
CMD ["jshell"]
