# Stage 1
FROM node:8-slim AS base

# set this with shell variables at build-time.
# If they aren't set, then not-set will be default.
ARG CREATED_DATE=not-set
ARG SOURCE_COMMIT=not-set

# labels from https://github.com/opencontainers/image-spec/blob/master/annotations.md
LABEL org.opencontainers.image.authors=robertmlowe@rmlowe.com
LABEL org.opencontainers.image.created=$CREATED_DATE
LABEL org.opencontainers.image.revision=$SOURCE_COMMIT
LABEL org.opencontainers.image.title="Sample Node.js Dockerfile with LABELS"
LABEL org.opencontainers.image.url=https://hub.docker.com/u/rmlowe
LABEL org.opencontainers.image.source=https://github.com/rmlowe/docker-mastery-for-nodejs
LABEL org.opencontainers.image.licenses=MIT
LABEL com.bretfisher.nodeversion=$NODE_VERSION

EXPOSE 80

ENV NODE_ENV=production

ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

RUN apt-get update && apt-get install curl -y

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

WORKDIR /app

COPY package*.json ./

RUN npm config list \
    && npm ci \
    && npm cache clean --force

CMD ["node", "server.js"]


# Stage 2
FROM base AS dev

ENV NODE_ENV=development

ENV PATH /app/node_modules/.bin:$PATH

# NOTE: these apt dependencies are only needed
# for testing. they shouldn't be in production
RUN apt-get update -qq && apt-get install -qy \ 
    ca-certificates \
    bzip2 \
    curl \
    libfontconfig \
    --no-install-recommends

RUN npm install --only=development

CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]

USER node


# Stage 3
FROM base AS test

COPY . .

ENV NODE_ENV=development

ENV PATH /app/node_modules/.bin:$PATH

COPY --from=dev /app/node_modules /app/node_modules

RUN npm test

# aqua microscanner, which needs a token for API access
# note this isn't super secret, so we'll use an ARG here
# https://github.com/aquasecurity/microscanner
# ARG MICROSCANNER_TOKEN
# ADD https://get.aquasec.com/microscanner /
# RUN chmod +x /microscanner
# RUN apk add --no-cache ca-certificates && update-ca-certificates
# RUN /microscanner $MICROSCANNER_TOKEN --continue-on-failure

CMD ["npm", "run", "int-test"]

USER node


# Stage 4
FROM base AS prod

COPY views server.js ./

USER node