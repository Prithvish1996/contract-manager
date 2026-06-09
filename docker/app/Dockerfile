FROM node:lts

RUN apt-get update && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g pnpm

WORKDIR /usr/src/app

COPY package.json ./

# Clone contract-consent-agent and patch the gitpkg URL to use local copy
RUN git clone --depth 1 \
      https://github.com/Prometheus-X-association/contract-consent-agent.git \
      /tmp/contract-consent-agent \
    && cd /tmp/contract-consent-agent/contract-agent \
    && npm install --legacy-peer-deps --ignore-scripts

RUN sed -i 's|https://gitpkg.now.sh/Prometheus-X-association/contract-consent-agent/contract-agent?v1.0.0|file:/tmp/contract-consent-agent/contract-agent|g' package.json

RUN rm -f pnpm-lock.yaml && npm install --legacy-peer-deps --ignore-scripts

COPY . .

RUN chown -R node:node /usr/src/app

RUN sed -i 's/"noEmitOnError"[[:space:]]*:[[:space:]]*true/"noEmitOnError": false/' tsconfig.json || true

RUN npm run build || true

CMD ["node", "build/start.js"]
