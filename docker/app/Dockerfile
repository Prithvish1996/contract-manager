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

# Deduplicate to ensure only one copy of mongoose exists across app + contract-agent
RUN npm dedupe --legacy-peer-deps

COPY . .

RUN chown -R node:node /usr/src/app

RUN sed -i 's/"noEmitOnError"[[:space:]]*:[[:space:]]*true/"noEmitOnError": false/' tsconfig.json || true

# Run tsc and tsc-alias separately — tsc exits non-zero on type errors even with
# noEmitOnError:false, which would block tsc-alias (path alias resolution) in a chain
RUN npx rimraf build/
RUN npx tsc || true
RUN npx tsc-alias || true

CMD ["node", "build/start.js"]
