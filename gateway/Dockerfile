FROM node:24.22.3-alpine3.24@sha256:116b800c3754b440ee0aa3ed2f20f5bf3d99a554bfd9adeea09ea8017f6e85f5 AS builder
WORKDIR /app
RUN apk add --no-cache python3 make g++

# Install gateway deps
COPY package.json package-lock.json tsconfig.json ./
RUN npm ci

# Install Angular frontend deps
COPY frontend/package.json frontend/package-lock.json ./frontend/
RUN npm ci --prefix frontend

# Copy source and build
COPY src ./src
COPY frontend/src ./frontend/src
COPY frontend/angular.json frontend/tsconfig*.json ./frontend/
RUN npm run build

# Remove devDependencies so only production node_modules remain
RUN npm prune --omit=dev


FROM node:24.22.3-alpine3.24@sha256:116b800c3754b440ee0aa3ed2f20f5bf3d99a554bfd9adeea09ea8017f6e85f5 AS runtime
WORKDIR /app
# The npm bundled in the node image ships vulnerable copies of undici and
# brace-expansion (they live inside npm itself, so no package.json override can reach
# them). Upgrading npm the supported way replaces them with fixed versions.
RUN npm install -g npm@11.21.2 && npm cache clean --force
RUN mkdir -p /data /tmp/dc-sockets
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package.json ./
COPY extensions ./extensions
ENV DB_PATH=/data/huddle.db
ENV EXT_DIR=/app/extensions
EXPOSE 80 3000
CMD ["node", "dist/index.js"]
