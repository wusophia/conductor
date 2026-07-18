#
# conductor:ui - Conductor UI
#
#FROM node:20-alpine
#LABEL maintainer="Orkes OSS <oss@orkes.io>"

# Install the required packages for the node build
# to run on alpine
#RUN apk update && apk add --no-cache python3 py3-pip make g++

# A directory within the virtualized Docker environment
# Becomes more relevant when using Docker Compose later
#WORKDIR /usr/src/app

# Copies package.json to Docker environment in a separate layer as a performance optimization
#COPY ./ui/package.json ./

# Installs all node packages. Cached unless package.json changes
#RUN yarn install && mkdir -p public && cp -r node_modules/monaco-editor public/

# Copies everything else over to Docker environment
# node_modules excluded in .dockerignore.
#COPY ./ui .

# Include monaco sources into bundle (instead of using CDN)
#ENV REACT_APP_MONACO_EDITOR_USING_CDN=false
#ENV REACT_APP_ENABLE_ERRORS_INSPECTOR=true
#CMD [ "yarn", "start" ]


#
# conductor:server - Build Conductor UI
#
# ===========================================================================================================
# 1. Builder stage 
# ===========================================================================================================
FROM alpine:3.20 AS ui-builder
ARG YARN_OPTS

LABEL maintainer="Orkes OSS <oss@orkes.io>"

# Install dependencies
RUN apk add --update nodejs npm yarn

COPY . /conductor
WORKDIR /conductor/ui

# Include monaco sources into bundle (instead of using CDN)
ENV REACT_APP_MONACO_EDITOR_USING_CDN=false
ENV REACT_APP_ENABLE_ERRORS_INSPECTOR=true
RUN yarn ${YARN_OPTS} install && cp -r node_modules/monaco-editor public/ && yarn ${YARN_OPTS} build
RUN ls -ltr
RUN echo "Done building UI"


# ===========================================================================================================
# 2. Bin stage
# ===========================================================================================================
FROM nginx:1.29.4-alpine

LABEL maintainer="Orkes OSS <oss@orkes.io>"

# Copy compiled UI assets to nginx www directory
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=ui-builder /conductor/ui/build .
COPY --from=ui-builder /conductor/docker/server/nginx/default.conf.template /etc/nginx/templates/default.conf.template

ENV NGINX_ENVSUBST_FILTER="CONDUCTOR_SERVER_URL"

CMD [ "nginx", "-g", "daemon off;" ]