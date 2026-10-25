# WindScout in a container, with its own docker daemon inside (Docker-in-Docker) for parallel scans, and its own
# Windscribe app (the Linux CLI app), so nothing is installed on your computer. Build straight from GitHub (or, in a clone, with . for the URL):
#   docker build -t windscout-sandbox -f sandbox/Dockerfile https://github.com/Diercohen/windscout.git
# Where Docker Hub refuses (Iran), take the base from a mirror: --build-arg BASE=registry.docker.ir/library/node:22-bookworm-slim
ARG BASE=node:22-bookworm-slim
FROM ${BASE}
ENV LANG=C.UTF-8
# Debian's mirrors and Windscribe's app as in the worker image (src/docker.ts); docker.io: Debian's own docker, for the daemon inside
RUN for m in deb.debian.org mirror.shatel.ir mirror.iranserver.com mirror.mobinhost.com mirror2.chabokan.net; do \
   sed -i -E "s#^URIs: [a-z]+://[^/]+/#URIs: http://$m/#" /etc/apt/sources.list.d/debian.sources; \
   apt-get -o Acquire::http::Timeout=20 update --error-on=any && break; \
 done \
 && apt-get install -y --no-install-recommends ca-certificates curl openresolv docker.io \
 && v=$(curl -fsSL -o /dev/null -w '%{url_effective}' https://github.com/Windscribe/Desktop-App/releases/latest | sed 's#.*/v##') \
 && curl -fsSL -o /tmp/windscribe.deb "https://github.com/Windscribe/Desktop-App/releases/download/v$v/windscribe-cli_"$v"_$(dpkg --print-architecture).deb" \
 && apt-get install -y --no-install-recommends /tmp/windscribe.deb \
 && rm -rf /tmp/windscribe.deb /var/lib/apt/lists/* \
 && useradd -m -G docker ws
WORKDIR /app
COPY package.json package-lock.json tsconfig.json ./
COPY src src
RUN npm ci && npm run build && npm prune --omit=dev && ln -s /app/dist/cli.js /usr/local/bin/windscout
COPY sandbox/entrypoint.sh /usr/local/bin/
# the inner docker's images and the workers' volumes, and the Windscribe login and windscout's settings: keep them in volumes
VOLUME /var/lib/docker /home/ws
ENTRYPOINT ["entrypoint.sh"]
