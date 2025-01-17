FROM node:20-alpine AS builder

# Install build dependencies
RUN apk add --no-cache python3 make g++ git

# Install pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

WORKDIR /app

# Copy only necessary files for build
COPY package.json pnpm-lock.yaml ./
COPY nx.json tsconfig.base.json ./

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build the application
RUN pnpm nx build api-gateway --prod

# Production stage - using smaller base image
FROM node:20-alpine AS runner

WORKDIR /app

# Copy only the built application and package files needed for production
COPY --from=builder /app/dist/apps/api-gateway ./

# Install only the absolutely necessary production dependencies
RUN npm install --production --no-package-lock tslib

# Clean npm cache
RUN npm cache clean --force

# Set environment variables
ENV NODE_ENV=production

# Expose the port the app runs on
EXPOSE 3000

# Start the application
CMD ["node", "main.js"]
