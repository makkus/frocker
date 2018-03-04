#! /usr/bin/env bash
#
# ========================================
#
# Wrapper to build/start/stop/etc Docker containers for freckle folders.
#
# Copyright: Markus Binsteiner, 2018
# License: GPL v3

# VARS
CONTAINER_NAME=wordpress
DOCKER_USERNAME="local"

FRECKLE_CONTEXT_REPO="frkl:${CONTAINER_NAME}"
FRECKELIZE_DOCKER_VARS_FILE=".freckelize/docker.yml"
FRECKLES_VERSION=git

MOUNT_MYSQL_VOLUME=false

# script start
# --------------

THIS_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd -P "$THIS_DIR"

MYSQL_DIR=".freckelize/docker/mysql_volume"
BACKUPS_DIR="backups"
WORDPRESS_DIR="wordpress"

IMAGE_NAME="${DOCKER_USERNAME}:${CONTAINER_NAME}"

#NO_CACHE="--no-cache"
NO_CACHE=""

if [[ "$1" == "--build" ]]; then
    docker build ${NO_CACHE} -t "$IMAGE_NAME" -f Dockerfile --build-arg FRECKLES_VERSION="${FRECKLES_VERSION}" --build-arg FRECKLE_CONTEXT_REPOS="-r ${FRECKLE_CONTEXT_REPO}" --build-arg FRECKLE_EXTRA_VARS="-v /freckle/${FRECKELIZE_DOCKER_VARS_FILE}" .  || { echo 'docker built process failed' ; exit 1; }
fi

if [ ! -d "${THIS_DIR}/${MYSQL_DIR}/mysql" ] && [ "$MOUNT_MYSQL_VOLUME" = true ]; then
    # copy 'raw' mysql database, so we don't loose the data we are working on when docker container is stopped
    echo "Copying initial mysql data to: ${THIS_DIR}/${MYSQL_DIR}..."
    mkdir -p "${THIS_DIR}/${MYSQL_DIR}"
    docker run -i --rm --mount type=bind,source="${THIS_DIR}/${MYSQL_DIR}",target="/mysql_init" ${IMAGE_NAME} /bin/bash << EOF

cp -a /var/lib/mysql/* /mysql_init

EOF
fi

if [ ! -s ${THIS_DIR}/wordpress/wp-config.php ]; then
    echo "Copying initial wordpress config file to: wordpress/wp-config.php..."
    docker run -i --rm --mount type=bind,source="${THIS_DIR}/wordpress",target="/freckle/wordpress_init" ${IMAGE_NAME} /bin/bash << EOF

cp -a /freckle/wordpress/wp-config.php /freckle/wordpress_init/wp-config.php

EOF
fi

if [ ! -d ${THIS_DIR}/wordpress/wp-content/themes ]; then
    echo "Copying initial wordpress content folder to: wordpress/wp-content..."
    docker run -i --rm --mount type=bind,source="${THIS_DIR}/wordpress",target="/freckle/wordpress_init" ${IMAGE_NAME} /bin/bash << EOF
  
cp -a /freckle/wordpress/wp-content/* /freckle/wordpress_init/wp-content/

EOF
fi

C_NAME="freckelize_${CONTAINER_NAME}"

result=$( docker ps | grep ${C_NAME} )

if [[ -n "$result" ]]; then
   echo "Container '${C_NAME}' already running. Doing nothing..."
   exit 0
fi

result=$( docker ps -a | grep ${C_NAME} )

if [[ -n "$result" ]]; then
   echo "Container '${C_NAME}' already exists. Starting..."
   docker start "${C_NAME}"
else

   if [ "$MOUNT_MYSQL_VOLUME" = true ]; then
       MYSQL_MOUNT_OPTION="--mount type=bind,source=\"${THIS_DIR}/${MYSQL_DIR}\",target=/var/lib/mysql"
   fi

   echo "Container '${C_NAME}' does not exist, creating and starting it..."
   docker_id=$(docker run --name "${C_NAME}" -p 8080:8080 ${MYSQL_MOUNT_OPTION} --mount type=bind,source="${THIS_DIR}",target="/freckle"  -d "$IMAGE_NAME")
   echo "   container id: ${docker_id}"
fi

exit 0


#docker run --rm -it -p 8080:8080 --mount type=bind,source="$THIS_DIR",target="/freckle" --mount type=bind,source="${THIS_DIR}/${MYSQL_DIR}",target="/var/lib/mysql"  "$IMAGE_NAME" /bin/bash --login
#docker run --rm -it -p 8080:8080 --mount type=bind,source="$THIS_DIR",target="/freckle" "$IMAGE_NAME" /bin/bash --login
#docker run -it -p 8080:8080 "$IMAGE_NAME" /bin/bash --login


