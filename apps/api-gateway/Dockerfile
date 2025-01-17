# -------------- BUILDER --------------
FROM node:alpine AS builder

# Add dependencies for node-gyp and other native modules
RUN apk add --no-cache python3 make g++ git openssh

WORKDIR /app

# Install pnpm globally
RUN npm install -g pnpm

# Copy workspace configuration files
COPY package.json pnpm-lock.yaml nx.json tsconfig.base.json .npmrc pnpm-workspace.yaml ./

# # Copy all package.json files
COPY apps/api-gateway/package.json ./apps/api-gateway/package.json

# Install all dependencies
RUN pnpm install -r

# Copy the rest of the source code
COPY apps/api-gateway ./apps/api-gateway

# Build all applications (or specific ones based on args)
ARG APP=api-gateway
RUN pnpm nx build ${APP} --prod

# -------------- RUNNER --------------
FROM node:alpine AS runner

WORKDIR /app


# Install pnpm globally
RUN npm install -g pnpm

# Copy workspace configuration
COPY --from=builder /app/package.json /app/pnpm-lock.yaml ./

# Copy the built application
ARG APP=api-gateway
COPY --from=builder /app/dist/apps/${APP} ./dist/apps/${APP}
# COPY --from=builder /app/apps/${APP}/package.json ./apps/${APP}/package.json

# Install production dependencies only
ENV NODE_ENV=production
RUN pnpm install --prod

# Start the application
CMD ["node", "dist/apps/api-gateway/main.js"] 