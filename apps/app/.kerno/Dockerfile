#
# Base image for the server
#
FROM node:18 AS base

ARG WORKSPACE_SCOPE=@conduit/app

# Set working directory
WORKDIR /app
RUN yarn global add turbo

# Copy the project files
COPY . .

# Use the build argument in the turbo prune command
RUN turbo prune --scope=$WORKSPACE_SCOPE --docker

#
# Installer image
# Add lockfile and package.json's of isolated sub-workspace
#
FROM node:18 AS installer

ARG WORKSPACE_SCOPE=@conduit/app

# Set working directory
WORKDIR /app

# Install turbo globally
RUN yarn global add turbo

# Copy necessary files to the installer stage
COPY .gitignore .gitignore
COPY --from=base /app/out/json/ .
COPY --from=base /app/out/yarn.lock ./yarn.lock

# Install dependencies
RUN yarn install

# Build the project using the variable scope
COPY --from=base /app/out/full/ .
RUN turbo run build --filter=$WORKSPACE_SCOPE...

#
# Runner image - production runtime
#
FROM node:18-slim AS runner

ARG WORKSPACE_SCOPE=@conduit/app

# Set working directory
WORKDIR /app

# Set production environment
ENV NODE_ENV=production
ENV NODE_CONFIG_DIR=../../config

# Copy built application and all dependencies from installer
COPY --from=installer /app .

# Expose port (default Express port)
EXPOSE 3000

# Set the default command to run the application
# The main entry point is index.js as specified in package.json
WORKDIR /app/apps/app
CMD ["node", "dist/index.js"]
