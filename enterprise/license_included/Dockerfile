# Liquibase Enterprise Release Version
# Enter the full version including the last 4 digits. For example, 7.18.7336
ARG LIQUIBASE_ENTERPRISE_VERSION=7.18.7336

# Builder Stage
FROM almalinux:8.8-minimal as builder

ARG LIQUIBASE_ENTERPRISE_VERSION

# Create Installer directory and download installation files
RUN \
    mkdir -p /opt/liquibase/installers && \
    mkdir -p /opt/liquibase/license && \
    curl https://download.liquibase.com/Datical_DB_Software/Datical_DB_${LIQUIBASE_ENTERPRISE_VERSION}/DaticalDB-linux.gtk.x86_64-${LIQUIBASE_ENTERPRISE_VERSION}.jar -o /opt/liquibase/installers/DaticalDB-linux.gtk.x86_64-${LIQUIBASE_ENTERPRISE_VERSION}.jar && \
    curl https://download.liquibase.com/Datical_DB_Software/Datical_DB_${LIQUIBASE_ENTERPRISE_VERSION}/DaticalDBCompositeRepo-${LIQUIBASE_ENTERPRISE_VERSION}.zip -o /opt/liquibase//installers/DaticalDBCompositeRepo-${LIQUIBASE_ENTERPRISE_VERSION}.zip

# Production Stage
FROM almalinux:8.8-minimal as production

ARG LIQUIBASE_ENTERPRISE_VERSION

# Install git, java, zip/unzip, database clients, and other dependencies
RUN \
    microdnf -y update && \
    ACCEPT_EULA=Y microdnf -y install \
    libaio \
    libnsl \
    git \
    java-11-openjdk-devel \
    zip \
    unzip && \
    #microdnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm && \
    #microdnf module disable -y postgresql && \
    #microdnf install -y postgresql14.x86_64 && \
    microdnf clean all

# Create liquibase user
RUN \
    groupadd liquibase && \
    useradd -d /opt/liquibase -g liquibase liquibase && \
    echo 'liquibase:liquibase'

# Setup installation directories
RUN \
    mkdir -p /opt/liquibase/installers && \
    mkdir -p /opt/liquibase/license && \
    chown -R liquibase /opt/liquibase

WORKDIR /opt/liquibase

# Change to the liquibase user
USER liquibase

# Copy from builder stage
COPY --from=builder --chown=liquibase:liquibase  /opt/liquibase/ /opt/liquibase/

# Copy autoinstall file to image
COPY --chown=liquibase:liquibase autoInstall.xml /opt/liquibase/installers

# Copy License File to image
COPY --chown=liquibase:liquibase myLicense.lic /opt/liquibase/license

# Install Liquibase Enterprise
RUN \
    set -x && \
    java -jar /opt/liquibase/installers/DaticalDB-linux.gtk.x86_64-${LIQUIBASE_ENTERPRISE_VERSION}.jar /opt/liquibase/installers/autoInstall.xml && \
    export PATH=$PATH:/opt/liquibase/repl && \
    echo "export PATH=$PATH:/opt/liquibase/repl" >> ~/.bashrc && \
    cp /opt/liquibase/license/*.lic /opt/liquibase/repl && \
    hammer installDrivers jar:file:/opt/liquibase/installers/DaticalDBCompositeRepo-${LIQUIBASE_ENTERPRISE_VERSION}.zip\!/ && \
    # Cleanup files to reduce the size of the image.
    rm /opt/liquibase/installers/DaticalDB-linux.gtk.x86_64-${LIQUIBASE_ENTERPRISE_VERSION}.jar && \
    rm /opt/liquibase/installers/DaticalDBCompositeRepo-${LIQUIBASE_ENTERPRISE_VERSION}.zip && \
    # Remove Oracle instant client if it is not needed
    # rm -rf /opt/liquibase/instantclient && \
    rm -rf /opt/liquibase/configuration && \
    rm -rf /opt/liquibase/plugins

# For Liquibase Enterprise Project Info
VOLUME /liquibase/project

# For Liquibase Packager using SQL Source Code Repo
VOLUME /liquibase/src

ENTRYPOINT ["/bin/bash"]