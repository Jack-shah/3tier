# ==========================================
# Stage 1: Build & Dependency Installation
# ==========================================
FROM node:18-alpine AS builder

# Set the working directory
WORKDIR /opt/server

# Copy package management files first to maximize Docker layer caching
COPY package.json package-lock.json* ./

RUN npm install

# Copy the rest of the application source code
COPY server.js ./
# Copy any other required application folders if present (e.g., public, routes)
# COPY images/ ./images/

# 2. PRUNE the node_modules to remove development-only packages
RUN npm prune --production

# ==========================================
# Stage 2: Secure Production Runtime
# ==========================================
# Using Google Distroless Node.js 18 image (contains no package managers, shells, or CVE baggage)
FROM gcr.io/distroless/nodejs18-debian11

# Set standard Node environment to production
ENV NODE_ENV=production

# Set the working directory inside distroless
WORKDIR /opt/server

# Copy the node_modules and source files from the builder stage
COPY --from=builder /opt/server/node_modules ./node_modules
COPY --from=builder /opt/server/server.js ./server.js
# COPY --from=builder /app/images ./images

# Expose the internal microservice port (Catalogue defaults to 8080 or customized via env)
EXPOSE 8091

# Distroless automatically executes the file using the node binary
CMD ["server.js"]
