# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

FROM opensuse/leap:15.0 as installer-env

ARG PS_VERSION=7.0.0-preview.1
ARG PS_PACKAGE=powershell-${PS_VERSION}-linux-x64.tar.gz
ARG PS_PACKAGE_URL=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE}
ARG PS_INSTALL_VERSION=7-preview

# Define ENVs for Localization/Globalization
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1 \
    # define the folder we will be installing PowerShell to
    PS_INSTALL_FOLDER=/opt/microsoft/powershell/$PS_INSTALL_VERSION \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-SUSE-15

# install dependencies
RUN mkdir -p ${PS_INSTALL_FOLDER} \
    && zypper install -y \
    curl \
    tar \
    gzip \
    && curl -sSL ${PS_PACKAGE_URL} -o /tmp/linux.tar.gz \
    && tar zxf /tmp/linux.tar.gz -C ${PS_INSTALL_FOLDER}

# start new stage so that the tar.gz layers are left behind in multi-stage build
FROM opensuse/leap:15.0 as final-image

ARG PS_VERSION=7.0.0-preview.1
ARG PS_INSTALL_VERSION=7-preview

# copy only files needed from the prior stage, others will be left behind
COPY --from=installer-env ["/opt/microsoft/powershell", "/opt/microsoft/powershell"]

# define ENVs for Localization/Globalization
ENV PS_INSTALL_VERSION=7.0.0-preview.1 \
    PS_INSTALL_FOLDER=/opt/microsoft/powershell/${PS_INSTALL_VERSION} \
    # set a fixed location for the Module analysis cache
    PSModuleAnalysisCachePath=/var/cache/microsoft/powershell/PSModuleAnalysisCache/ModuleAnalysisCache \
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-CentOS-7.3.1611 \
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1

# install dependencies and clean up
RUN zypper install -y \
    # these libraries were listed as dependencies when installing .rpm powershell package
    libicu60_2 \
    libicu60_2-ledata \
    # give all user execute permissions and remove write permissions for others
    && chmod a+x,o-w ${PS_INSTALL_FOLDER}/pwsh \
    # create pwsh symbolic link pointing to powershell
    && ln -sf ${PS_INSTALL_FOLDER}/pwsh /usr/bin/pwsh \
    # Create the pwsh preview symbolic link that points to powershell
    && ln -sf ${PS_INSTALL_FOLDER}/pwsh /usr/bin/pwsh-preview \
    # intialize powershell module cache
    && export POWERSHELL_TELEMETRY_OPTOUT=1 \
    # and disable telemetry
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
CMD [ "pwsh-preview" ]
