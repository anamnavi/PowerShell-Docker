# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

FROM centos:8 AS installer-env

ARG PS_VERSION=7.0.0
ARG PACKAGE_VERSION=7.0.0
ARG PS_PACKAGE=powershell-${PACKAGE_VERSION}-1.rhel.7.x86_64.rpm
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}

# Define ENVs for Localization/Globalization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-CentOS-8

# Install dependencies and clean up
RUN curl -sSL ${PS_PACKAGE_URL} -o /tmp/powershell.rpm \
    && yum install -y /tmp/powershell.rpm \
    # Required for gssntlmssp
    && yum install -y epel-release \
    # Update now that we have epel-release
    && yum update -y \
    # Install libraries for NTLM support
    && yum install -y \
      gssntlmssp \
      # less is required for help in powershell
      less \
      # clear is part of ncurses which is used by clear
      ncurses \
      # PowerShell remoting over SSH dependencies
      openssh-clients \
    && yum upgrade-minimal -y --security \
    && yum clean all \
    # remove powershell package
    && rm /tmp/powershell.rpm \
    && ln -sf /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh \
    # intialize powershell module cache
    # and disable telemetry
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    && pwsh \
        -NoLogo \
        -NoProfile \
        -Command " \
          \$ErrorActionPreference = 'Stop' ; \
          \$ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path \$env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            Start-Sleep -Seconds 6 ; \
          }"

# Use PowerShell as the default shell
# Use array to avoid Docker prepending /bin/sh -c
CMD [ "pwsh" ]
