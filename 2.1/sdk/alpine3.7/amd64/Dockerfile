ARG REPO=microsoft/dotnet
FROM $REPO:2.1-runtime-deps-alpine3.7


ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # Install .NET Core SDK
    DOTNET_SDK_VERSION=2.1.504 \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \ 
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip \
    JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk/jre \
    PATH=$PATH:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin:/root/.dotnet/tools \
    JAVA_VERSION=8u181 \
    JAVA_ALPINE_VERSION=8.181.13-r0

# Disable the invariant mode (set in base image) and install openjdk
RUN apk add --no-cache icu-libs openjdk8-jre="$JAVA_ALPINE_VERSION"

RUN apk add --no-cache --virtual .build-deps \
        openssl \
    && wget -O dotnet.tar.gz https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$DOTNET_SDK_VERSION/dotnet-sdk-$DOTNET_SDK_VERSION-linux-musl-x64.tar.gz \
    && dotnet_sha512='fec149ca27513d441ed1ec57fa690a6bc92d45d5e034388d6fb3c7577cbdbb2d38e5c6c3702c91ce9772d651b0cfbf5ce9d2446a555d89d7402939b9ce6b4653' \
    && echo "$dotnet_sha512  dotnet.tar.gz" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet \
    && tar -C /usr/share/dotnet -xzf dotnet.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
    && rm dotnet.tar.gz \
    && apk del .build-deps \
    # install tools
    && dotnet tool install --global coverlet.console \
    && dotnet tool install --global dotnet-sonarscanner
    
# add a simple script that can auto-detect the appropriate JAVA_HOME value
# based on whether the JDK or only the JRE is installed
RUN set -x \
    && { \
		echo '#!/bin/sh'; \
		echo 'set -e'; \
		echo; \
		echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"'; \
	} > /usr/local/bin/docker-java-home \
	&& chmod +x /usr/local/bin/docker-java-home \
	&& [ "$JAVA_HOME" = "$(docker-java-home)" ]
    
RUN dotnet help
