# Ponderada Deploying Backstage - M9 S2

## Passo a passo para compilação e execução do Backstage com Docker

### Instalar o backstage:
1º Rode o comando `npx @backstage/create-app@latest --skip-install`;

2º Defina um nome para a sua aplicação;

3º Após criar a aplicação, navegue até a pasta da aplicação e rode o comando `yarn install` (caso o seu yarn esteja desatualizado, rodar o comando `npm install --global yarn`);

### Preparar o build do backstage:
4º Rode o comando `yarn install --frozen-lockfile`;

5º Prepare os types com o comando `yarn tsc`;

6º Rode o comando `yarn build:backend` para buildar o backend;

### Ajustar o Dockerfile do backstage:
7º Abra o projeto no VSCode;

8º Acesse o arquivo _Dockerfile_ do backend navegando por: _packages_ > _backend_ > _Dockerfile_;

9º Substitua todo o conteúdo do arquivo pelo código abaixo:

````
FROM node:18-bookworm-slim

# Install isolate-vm dependencies, these are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# From here on we use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
# If it is instead created as `root` then the `tar` command below will
# fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV development

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
````

## Imagens da execução dos passos:
![1](https://github.com/liviabonotto/ponds-s2-m10/assets/87043469/f108bd3f-537c-47b6-a619-1e7a8a930e9d)
![2](https://github.com/liviabonotto/ponds-s2-m10/assets/87043469/9ed34af6-4cc4-47d1-8599-51f684afff6c)
![3](https://github.com/liviabonotto/ponds-s2-m10/assets/87043469/f5b5787e-9dee-4ca0-9b9d-1f50272e1bbe)


## Painel do Backstage:
![4](https://github.com/liviabonotto/ponds-s2-m10/assets/87043469/1a455a6a-2390-4545-9af7-ddcb3038861a)
![5](https://github.com/liviabonotto/ponds-s2-m10/assets/87043469/8bc9280e-4a12-4d76-bcaf-65840129dfad)

