ARG BASE_IMAGE
FROM $BASE_IMAGE
#USER root
# Add again root CA with `update-ca-certificates` tool
#RUN apk --no-cache add ca-certificates \
#    && rm -rf /var/cache/apk/*

USER 1001
ENV DEX_FRONTEND_DIR=/srv/dex/web
COPY --chown=root:root web /srv/dex/web
