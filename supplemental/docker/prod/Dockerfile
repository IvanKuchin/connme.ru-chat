FROM ubuntu:xenial as base

ARG BUILD_REPO
ARG MAIN_SUBDOMAIN=www
ARG IMAGE_SUBDOMAIN=images
ARG ROOTDIR=/home/httpd/www
ARG IMAGEROOTDIR=/home/httpd/images
ARG LOCAL_INSTALL_DIR=/usr/local/share/www
ARG BUILD_RELEASE_NUMBER=0.0.1

ARG GIT_FOLDER=/backend

LABEL maintainer="ivan.kuchin@gmail.com"
LABEL org.opencontainers.image.source="https://github.com/IvanKuchin/${BUILD_REPO}"

###########################################
# check if all mandatory args are defined #
###########################################
RUN [ -z "$BUILD_REPO" ] && echo "BUILD_REPO is required" && exit 1 || true
RUN echo repository is $BUILD_REPO

###########################
# install basic utilities #
###########################
RUN apt update \
    && apt install --no-install-recommends --no-install-suggests -y \
           ca-certificates \
           tzdata \
           locales \
           gettext \
    && rm -rf /var/lib/apt/lists/*

####################
# install timezone #
####################
#RUN apt-get install --no-install-recommends --no-install-suggests -y tzdata \
#    && rm -rf /var/lib/apt/lists/*
RUN cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime
RUN echo "Europe/Moscow" > /etc/timezone
RUN date

##############################
# install additional locales #
##############################
#RUN apt-get install --no-install-recommends --no-install-suggests -y locales gettext 
#    && rm -rf /var/lib/apt/lists/*
RUN locale-gen ru_RU.utf8
RUN locale-gen en_US.utf8
RUN update-locale LC_ALL=en_US.UTF8
# RUN export LC_ALL=en_US.utf8

############################################################################################


FROM base as build

ARG BUILD_REPO
ARG MAIN_SUBDOMAIN
ARG IMAGE_SUBDOMAIN
ARG ROOTDIR
ARG IMAGEROOTDIR
ARG LOCAL_INSTALL_DIR

########################
# install dependencies #
########################
RUN echo 123456
RUN apt update  \
    # && apt-get install --no-install-recommends --no-install-suggests -y software-properties-common \
    # && add-apt-repository -y ppa:jonathonf/ffmpeg-4 \
    # && apt update \
    && apt-get install --no-install-recommends --no-install-suggests -y \ 
        build-essential \
        file \
        cmake \
        gsfonts \
        graphicsmagick-libmagick-dev-compat \
        # libavcodec-dev \
        # libavformat-dev \
        # libavutil-dev \
        libcurl4-openssl-dev \
        libgraphicsmagick++-dev \
        # libhpdf-dev \
        # libmaxminddb-dev \
        libmysqlclient-dev \
        libssl-dev \
        libwebp-dev \
        libarchive-dev \
        rapidjson-dev \
        pkg-config \
        git

#############################
# libwebsocket installation #
#############################
RUN git clone -b v2.2.0 --depth 1 https://github.com/warmcat/libwebsockets.git /libwebsockets
WORKDIR /libwebsockets
RUN mkdir build \
 && cd build \
 && cmake -DLWS_WITHOUT_DAEMONIZE:BOOL=FALSE -DLWS_WITH_STATIC=1 -DLWS_WITH_SHARED=1 -DCMAKE_INSTALL_PREFIX:PATH=/usr/local/share/libwebsockets/ .. \
 && make -j2 install && make clean

############
# pull src #
############
COPY . ${GIT_FOLDER}
WORKDIR ${GIT_FOLDER}
RUN git clone --depth 1 https://github.com/IvanKuchin/backend_pi.git ./src/pi

##################
# save commit id #
##################
RUN git rev-parse HEAD > ./supplemental/git/git_commit_id

##################### 
# build from source #
##################### 
RUN mkdir build \
    && cd build \ 
    && cmake \
        -DPROJECT_NAME=${BUILD_REPO} \
        -DMAIN_SUBDOMAIN=${MAIN_SUBDOMAIN} \
        -DIMAGE_SUBDOMAIN=${IMAGE_SUBDOMAIN} \
        -DROOTDIR=${ROOTDIR} \
        -DIMAGEROOTDIR=${IMAGEROOTDIR} \
        -DLOCAL_INSTALL_DIR=${LOCAL_INSTALL_DIR} \
        -DBUILD_RELEASE_NUMBERS=${BUILD_RELEASE_NUMBERS} \
        -DFFPMPEG_DISABLE=true \
        -DMAXMIND_DISABLE=true \
        .. \
    && make -j2 package

############################################################################################

FROM base as app

ARG BUILD_REPO
ARG MAIN_SUBDOMAIN
ARG IMAGE_SUBDOMAIN
ARG ROOTDIR
ARG IMAGEROOTDIR
ARG LOCAL_INSTALL_DIR

WORKDIR ${GIT_FOLDER}

####################
# copy from build  #
####################
COPY --from=build ${GIT_FOLDER}/build/${BUILD_REPO}-infra.deb ./
COPY --from=build ${GIT_FOLDER}/build/${BUILD_REPO}-chat.deb ./


####################
# install binaries #
####################
RUN apt update \
    && apt install  --no-install-recommends --no-install-suggests -y ./*.deb \
    && rm -rf /var/lib/apt/lists/*

WORKDIR ${LOCAL_INSTALL_DIR}/chat
CMD ["./chat-server", "-s"]

