FROM jetbrains/teamcity-agent:2020.1.4

USER root

# Cleanup previous sdk 3.1.401
RUN rm /usr/bin/dotnet && rm -rf /usr/share/dotnet

# from 3.1 bionic; 
# https://github.com/JetBrains/teamcity-docker-agent/blob/master/ubuntu/Dockerfile
# jetbrains/teamcity-agent:2020.1.4 already have some
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libssl1.1 \
    && rm -rf /var/lib/apt/lists/*

# 3.1.402: https://github.com/dotnet/dotnet-docker/blob/master/src/sdk/3.1/bionic/amd64/Dockerfile
# 2.2.207: https://github.com/dotnet/dotnet-docker/blob/1fff14a1bd042cd5c5fa760d8c5fbb2cbf2be171/2.2/sdk/bionic/amd64/Dockerfile
# 2.1.810: https://github.com/dotnet/dotnet-docker/blob/9f916e26dcc8fcd24193941cf9b5125c521d895f/src/sdk/2.1/bionic/amd64/Dockerfile

# Install multi .NET Core SDK
ENV DOTNET_SDK_VERSION21=2.1.810 \
    DOTNET_SDK_VERSION22=2.2.207 \
    DOTNET_SDK_VERSION31=3.1.402 \
    DOTNET_SDK_VERSION=3.1.402 \
    # Configure web servers to bind to port 80 when present
    # ASPNETCORE_URLS=http://+:80 \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip

RUN curl -SL --output dotnet21.tar.gz https://dotnetcli.azureedge.net/dotnet/Sdk/$DOTNET_SDK_VERSION21/dotnet-sdk-$DOTNET_SDK_VERSION21-linux-x64.tar.gz \
    && dotnet_sha512='45ceacbf393778254c5e95184d1c284ea21beb38b1610803390858960e5edb1faee0f0c1355e1306ea282f864645fb38fccf85cc01fb10d8ebf48f2b6a658bd0' \
    && echo "$dotnet_sha512 dotnet21.tar.gz" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet \
    && tar -zxf dotnet21.tar.gz -C /usr/share/dotnet \
    && rm dotnet21.tar.gz \
    # && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
    && curl -SL --output dotnet22.tar.gz https://dotnetcli.azureedge.net/dotnet/Sdk/$DOTNET_SDK_VERSION22/dotnet-sdk-$DOTNET_SDK_VERSION22-linux-x64.tar.gz \
    && dotnet_sha512='9d70b4a8a63b66da90544087199a0f681d135bf90d43ca53b12ea97cc600a768b0a3d2f824cfe27bd3228e058b060c63319cd86033be8b8d27925283f99de958' \
    && echo "$dotnet_sha512 dotnet22.tar.gz" | sha512sum -c - \
    # && mkdir -p /usr/share/dotnet \
    && tar -zxf dotnet22.tar.gz -C /usr/share/dotnet \
    && rm dotnet22.tar.gz \
    # && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
    && curl -SL --output dotnet31.tar.gz https://dotnetcli.azureedge.net/dotnet/Sdk/$DOTNET_SDK_VERSION31/dotnet-sdk-$DOTNET_SDK_VERSION31-linux-x64.tar.gz \
    && dotnet_sha512='42154efb5ad66ae3dcc300b2c0573a9537dd916fc48cbae92885a63a0b6d7f7c3a4366ca2298107783bc1f1913328f35e778dcda378da276cff3b8269495d5be' \
    && echo "$dotnet_sha512 dotnet31.tar.gz" | sha512sum -c - \
    # && mkdir -p /usr/share/dotnet \
    && tar -zxf dotnet31.tar.gz -C /usr/share/dotnet \
    && rm dotnet31.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet

# Trigger first run experience by running arbitrary cmd
RUN dotnet help

# https://github.com/nodejs/docker-node/blob/master/12/buster/Dockerfile
ENV NODE_VERSION=12.18.3 \
    NPM_VERSION=6.14.6

RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
  && case "${dpkgArch##*-}" in \
    amd64) ARCH='x64';; \
    ppc64el) ARCH='ppc64le';; \
    s390x) ARCH='s390x';; \
    arm64) ARCH='arm64';; \
    armhf) ARCH='armv7l';; \
    i386) ARCH='x86';; \
    *) echo "unsupported architecture"; exit 1 ;; \
  esac \
  # gpg keys listed at https://github.com/nodejs/node#release-keys
  && set -ex \
  && for key in \
    4ED778F539E3634C779C87C6D7062848A1AB005C \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
    B9E2F5981AA6E0CD28160D9FF13993A75599653C \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
  && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
  && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
  # smoke tests
  && node --version \
  && npm --version

ENV YARN_VERSION 1.22.4

RUN set -ex \
  && for key in \
    6A010C5166006599AA17F08146C2130DFD2497F5 \
  ; do \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
  && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
  && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  && mkdir -p /opt \
  && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn \
  && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg \
  && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
  # smoke test
  && yarn --version

# Install PowerShell global tool
# Install Coverlet Console global tool
# Install SonarScanner for .Net Core 2.1 global tool
# Install ReportGenerator global tool
ENV POWERSHELL_VERSION=7.0.3 \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetCoreSDK-Ubuntu-18.04 \
    DOTNET_COVERLET_VERSION=1.7.2 \
    DOTNET_SONARSCANNER_VERSION=4.10.0 \
    DOTNET_REPORTGENERATOR_VERSION=4.6.7

RUN curl -SL --output PowerShell.Linux.x64.$POWERSHELL_VERSION.nupkg https://pwshtool.blob.core.windows.net/tool/$POWERSHELL_VERSION/PowerShell.Linux.x64.$POWERSHELL_VERSION.nupkg \
    && powershell_sha512='580f405d26df40378f3abff3ec7e4ecaa46bb0e46bcb2b3c16eff2ead28fde5aaa55c19501f73315b454e68d98c9ef49f8887c36e7c733d7c8ea3dd70977da2f' \
    && echo "$powershell_sha512  PowerShell.Linux.x64.$POWERSHELL_VERSION.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/powershell \
    && dotnet tool install --add-source / --tool-path /usr/share/powershell --version $POWERSHELL_VERSION PowerShell.Linux.x64 \
    # && dotnet nuget locals all --clear \
    && rm PowerShell.Linux.x64.$POWERSHELL_VERSION.nupkg \
    && ln -s /usr/share/powershell/pwsh /usr/bin/pwsh \
    && chmod 755 /usr/share/powershell/pwsh \
    # To reduce image size, remove the copy nupkg that nuget keeps.
    && find /usr/share/powershell -print | grep -i '.*[.]nupkg$' | xargs rm \
    && curl -SL --output coverlet.console.$DOTNET_COVERLET_VERSION.nupkg https://globalcdn.nuget.org/packages/coverlet.console.$DOTNET_COVERLET_VERSION.nupkg \
    && coverlet_sha512='29632928f65e4019f22d4d6014cdc7aa123e50263c43d146e5599b82c074bea8165d73a786171738c7600872186c370d22f49292bc147d0c48997c5b1ab4e779' \
    && echo "$coverlet_sha512  coverlet.console.$DOTNET_COVERLET_VERSION.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/coverlet.console \
    && dotnet tool install --add-source / --tool-path /usr/share/coverlet.console --version $DOTNET_COVERLET_VERSION coverlet.console \
    && rm coverlet.console.$DOTNET_COVERLET_VERSION.nupkg \
    && ln -s /usr/share/coverlet.console/coverlet /usr/bin/coverlet \
    && chmod 755 /usr/share/coverlet.console/coverlet \
    && find /usr/share/coverlet.console -print | grep -i '.*[.]nupkg$' | xargs rm \
    && curl -SL --output dotnet-sonarscanner.$DOTNET_SONARSCANNER_VERSION.nupkg https://globalcdn.nuget.org/packages/dotnet-sonarscanner.$DOTNET_SONARSCANNER_VERSION.nupkg \
    && dotnet_sonarscanner_sha512='8e947712d30f47840a02349b92c93cbfe0174b9291683ea373b3ab7584edc27ad97a95a1405c44538a2a3f9fb6311b2efd6c4f24e0d171ba3e73dfd4746e8265' \
    && echo "$dotnet_sonarscanner_sha512  dotnet-sonarscanner.$DOTNET_SONARSCANNER_VERSION.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet-sonarscanner \
    && dotnet tool install --add-source / --tool-path /usr/share/dotnet-sonarscanner --version $DOTNET_SONARSCANNER_VERSION dotnet-sonarscanner \
    && rm dotnet-sonarscanner.$DOTNET_SONARSCANNER_VERSION.nupkg \
    && ln -s /usr/share/dotnet-sonarscanner/dotnet-sonarscanner /usr/bin/dotnet-sonarscanner \
    && chmod 755 /usr/share/dotnet-sonarscanner/dotnet-sonarscanner \
    && find /usr/share/dotnet-sonarscanner -print | grep -i '.*[.]nupkg$' | xargs rm \
    && curl -SL --output dotnet-reportgenerator-globaltool.$DOTNET_REPORTGENERATOR_VERSION.nupkg https://globalcdn.nuget.org/packages/dotnet-reportgenerator-globaltool.$DOTNET_REPORTGENERATOR_VERSION.nupkg \
    && reportgenerator_sha512='a888c246df99e56e60bb460dcf27e1ef37865f4d0b5524a6edf297cee44ec65d22e87a519095cdb59ec1f012d8c43d6c2794a86b2b0b7a0b2126e13abe17c058' \
    && echo "$reportgenerator_sha512  dotnet-reportgenerator-globaltool.$DOTNET_REPORTGENERATOR_VERSION.nupkg" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet-reportgenerator-globaltool \
    && dotnet tool install --add-source / --tool-path /usr/share/dotnet-reportgenerator-globaltool --version $DOTNET_REPORTGENERATOR_VERSION dotnet-reportgenerator-globaltool \
    && rm dotnet-reportgenerator-globaltool.$DOTNET_REPORTGENERATOR_VERSION.nupkg \
    && ln -s /usr/share/dotnet-reportgenerator-globaltool/reportgenerator /usr/bin/reportgenerator \
    && chmod 755 /usr/share/dotnet-reportgenerator-globaltool/reportgenerator \
    && find /usr/share/dotnet-reportgenerator-globaltool -print | grep -i '.*[.]nupkg$' | xargs rm \
    && dotnet nuget locals all --clear

USER buildagent

# ENV  PATH="$PATH:/home/buildagent/.dotnet/tools"
# RUN dotnet tool install --global coverlet.console --version $DOTNET_COVERLET_VERSION \
#     && dotnet tool install --global dotnet-sonarscanner --version $DOTNET_SONARSCANNER_VERSION \
#     && dotnet tool install --global dotnet-reportgenerator-globaltool --version $DOTNET_REPORTGENERATOR_VERSION

# Install module Pester
RUN pwsh -Command "Install-Module -Name Pester -Force"