# Defense Builders SDK - Base Image
# Secure, hardened foundation for all DBSDK containers
# https://github.com/iotactical/defense-builders-sdk

FROM ubuntu:22.04

LABEL org.opencontainers.image.source="https://github.com/iotactical/defense-builders-sdk"
LABEL org.opencontainers.image.description="Defense Builders SDK Base Image"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="ioTACTICAL <engineering@iotactical.co>"

# Environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV DBSDK_VERSION=1.0.0
ENV DBSDK_BUILD_DATE=${BUILD_DATE:-}
ENV DBSDK_GIT_SHA=${GIT_SHA:-}

# Install base security and development tools
RUN apt-get update && apt-get install -y \
    # Security essentials
    ca-certificates \
    gnupg \
    curl \
    wget \
    # Development essentials
    git \
    build-essential \
    software-properties-common \
    apt-transport-https \
    # Networking and utilities
    net-tools \
    iputils-ping \
    # Text processing
    jq \
    vim \
    nano \
    # Process management
    htop \
    procps \
    # UUID generation for telemetry
    uuid-runtime \
    # Clean up
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/*

# Create non-root user for security
RUN groupadd -g 1000 vscode && \
    useradd -u 1000 -g 1000 -m -s /bin/bash vscode && \
    echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set up workspace directory
RUN mkdir -p /workspaces && chown -R vscode:vscode /workspaces

# Install TAK Design System
ARG TAK_DESIGN_SYSTEM_VERSION=0.1.0
ENV TAK_DESIGN_SYSTEM_PATH=/opt/tak-design-system
ENV TAK_DESIGN_SYSTEM_VERSION=${TAK_DESIGN_SYSTEM_VERSION}
RUN mkdir -p ${TAK_DESIGN_SYSTEM_PATH} && \
    curl -fsSL "https://github.com/iotactical/tak-design-system/releases/download/v${TAK_DESIGN_SYSTEM_VERSION}/tak-design-system-v${TAK_DESIGN_SYSTEM_VERSION}.tar.gz" \
    -o /tmp/tak-design-system.tar.gz && \
    tar -xzf /tmp/tak-design-system.tar.gz -C ${TAK_DESIGN_SYSTEM_PATH} && \
    rm /tmp/tak-design-system.tar.gz && \
    chown -R vscode:vscode ${TAK_DESIGN_SYSTEM_PATH}
ENV TAK_DESIGN_ATAK_RES=${TAK_DESIGN_SYSTEM_PATH}/platforms/atak/res

# Install DBSDK telemetry and security tools
COPY base/telemetry/ /opt/dbsdk/telemetry/
COPY base/security/ /opt/dbsdk/security/
COPY base/scripts/ /opt/dbsdk/scripts/

# Make scripts executable
RUN chmod +x /opt/dbsdk/scripts/* && \
    chmod +x /opt/dbsdk/telemetry/* && \
    chmod +x /opt/dbsdk/security/*

# Install DBSDK utility in system PATH
RUN ln -s /opt/dbsdk/scripts/dbsdk /usr/local/bin/dbsdk

# Install telemetry service (privacy-first)
RUN /opt/dbsdk/scripts/install-telemetry.sh

# Run security hardening
RUN /opt/dbsdk/scripts/harden-container.sh

# Generate Software Bill of Materials (SBOM)
RUN /opt/dbsdk/security/generate-sbom.sh

# Switch to non-root user
USER vscode
WORKDIR /workspaces

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD /opt/dbsdk/scripts/healthcheck.sh

# Default command
CMD ["/bin/bash"]

# Build metadata
ARG BUILD_DATE
ARG GIT_SHA
LABEL org.opencontainers.image.created=${BUILD_DATE}
LABEL org.opencontainers.image.revision=${GIT_SHA}