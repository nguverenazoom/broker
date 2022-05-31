ARG NODEJS_VERSION=16.15.0
ARG NODEJS_MAJOR_VERSION=16

# --------------------------------------------------------
# 1) Node JS & broker Installation
# --------------------------------------------------------
FROM registry.access.redhat.com/ubi8/ubi:8.6 as base

ARG NODEJS_VERSION
ARG NODEJS_MAJOR_VERSION

# Download and extract Node
WORKDIR /tmp
RUN curl -o "/tmp/node-v${NODEJS_VERSION}-linux-x64.tar.gz" "https://nodejs.org/dist/v${NODEJS_VERSION}/node-v${NODEJS_VERSION}-linux-x64.tar.gz"
RUN tar xvf "/tmp/node-v${NODEJS_VERSION}-linux-x64.tar.gz"

# Install npm for the snyk-broker installation
RUN dnf module enable -y nodejs:${NODEJS_MAJOR_VERSION}
RUN yum install -y npm

# Install CA certificates
RUN yum install ca-certificates

ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
ENV PATH=$PATH:/home/node/.npm-global/bin

# Globally install snyk-broker
RUN npm install --global snyk-broker

# --------------------------------------------------------
# 2) Setting final image with Node and broker binary files
# --------------------------------------------------------

FROM registry.access.redhat.com/ubi8/ubi:8.6

ARG NODEJS_VERSION

ENV PATH=$PATH:/home/node/.npm-global/bin

# Copy the broker installation and CA certificates
COPY --from=base /home/node/.npm-global /home/node/.npm-global
COPY --from=base /etc/pki/tls/certs/ca-bundle.crt /etc/pki/tls/certs/ca-bundle.crt

# Copy node binary and set path
COPY --from=base "/tmp/node-v${NODEJS_VERSION}-linux-x64/bin/node" "/usr/node-v${NODEJS_VERSION}-linux-x64/bin/"
ENV PATH=/usr/node-v${NODEJS_VERSION}-linux-x64/bin:${PATH}

# Adding the node user
RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

RUN chown node:node /home/node

WORKDIR /home/node

# Don't run as root
USER node

# Generate default accept filter
RUN broker init bitbucket-server

######################################
# Custom Broker Client configuration #
# Redefine in derived Dockerfile,    #
# or provide as runtime args `-e`    #
######################################

# Your unique broker identifier
# ENV BROKER_TOKEN <broker-token>

# Your personal username to your bitbucket server account
# ENV BITBUCKET_USERNAME <username>

# Your personal password to your bitbucket server account
# ENV BITBUCKET_PASSWORD <password>

# Your Bitbucket Server host, excluding scheme
# ENV BITBUCKET your.bitbucket.server.hostname

# The Bitbucket server API URL
# ENV BITBUCKET_API $BITBUCKET/rest/api/1.0

# The port used by the broker client to accept internal connections
# Default value is 7341
# ENV PORT 7341

# The URL of your broker client (including scheme and port)
# This will be used as the webhook payload URL coming in from
# your Bitbucket Server.
# ENV BROKER_CLIENT_URL http://<broker.client.hostname>:$PORT

EXPOSE $PORT

CMD ["broker", "--verbose"]
