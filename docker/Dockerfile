FROM alpine:latest as builder

# version checksum of the archive to download
ARG VERSION
ARG SHA256

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]
RUN set -ox pipefail \
    && if [[ "${VERSION}" == "" ]]; then \
        echo "build-arg VERSION is required" \
        && exit 1; \
    fi \
    && if [[ "${SHA256}" == "" ]]; then \
        echo "build-arg SHA256 is required" \
        && exit 1; \
    fi \
    && archive="/tmp/factorio_headless_x64_$VERSION.tar.xz" \
    && apk add --update --no-cache --no-progress curl \
    && curl -sSL "https://www.factorio.com/get-download/$VERSION/headless/linux64" -o "$archive" \
    && echo "$SHA256  $archive" | sha256sum -c \
    || (sha256sum "$archive" && file "$archive" && exit 1) \
    && tar xf "$archive" --directory /opt \
    && rm "$archive"

FROM frolvlad/alpine-glibc:alpine-3.12
LABEL maintainer="https://github.com/factoriotools/factorio-docker"

ARG GROUP=factorio
ARG USER=factorio
ARG PGID=845
ARG PUID=845
ENV PORT=34197 \
    RCON_PORT=27015 \
    SAVES=/factorio/saves \
    CONFIG=/factorio/config \
    MODS=/factorio/mods \
    SCENARIOS=/factorio/scenarios \
    SCRIPTOUTPUT=/factorio/script-output \
    PUID="$PUID" \
    PGID="$PGID"

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]
RUN set -ox pipefail \
    && apk add --update --no-cache --no-progress \
        bash \
        curl \
        jq \
        pwgen \
        shadow \
        su-exec \
    && addgroup -g "$PGID" -S "$GROUP" \
    && adduser -u "$PUID" -G "$GROUP" -s /bin/sh -SDH "$USER" \
    && mkdir /factorio \
    && chown -R "$USER":"$GROUP" /factorio

# TODO merge back into above so tools get updates when rebuilt
COPY --from=builder --chown=$PUID:$PGID /opt/factorio /opt/factorio
USER $USER:$GROUP
RUN ln -s "$SCENARIOS" /opt/factorio/scenarios \
    && ln -s "$SAVES" /opt/factorio/saves \
    && mkdir /opt/factorio/config/

USER root:root
COPY files/*.sh /
COPY --chown=$PUID:&GID files/config.ini /opt/factorio/config/config.ini

VOLUME /factorio
EXPOSE $PORT/udp $RCON_PORT/tcp
ENTRYPOINT ["/docker-entrypoint.sh"]
