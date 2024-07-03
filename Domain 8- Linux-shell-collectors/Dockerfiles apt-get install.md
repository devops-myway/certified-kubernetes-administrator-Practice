#####  Dockerfiles - apt-get Install
https://github.com/nextcloud/docker/blob/424364e2e10a9d6e1a31e6659e2149aac1f1c772/12.0/apache/Dockerfile?spm=a2c65.11461447.0.0.47cd3b74HeBjnW

``````sh
  RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        rsync \
        bzip2 \
        busybox-static \
    ; \
    rm -rf /var/lib/apt/lists/*; \
``````

#####  

``````sh
RUN apt-get update && \
    apt-get install -y \
      curl procps \
  && rm -rf /var/lib/apt/lists/* 
``````
