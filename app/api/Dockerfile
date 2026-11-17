
# Stage 1: Install production dependencies
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev --no-audit --no-fund \
    && npm cache clean --force

COPY . .


# Stage 2: Lightweight production image
FROM alpine:3.24

# Install Node.js runtime libraries
RUN apk add --no-cache libstdc++ libgcc

# Copy Node.js runtime and npm
COPY --from=build /usr/local/bin/node /usr/local/bin/node
COPY --from=build /usr/local/lib/node_modules/npm /usr/local/lib/node_modules/npm

# Make npm available
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm

WORKDIR /app

# Copy application and production dependencies
COPY --from=build --chown=1000:1000 /app /app

ENV NODE_ENV=production

# Run as non-root user
USER 1000:1000

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD node -e "fetch('http://127.0.0.1:3000/api/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"

CMD ["npm", "start"]
