#
# Base image for the server
#
FROM node:18 AS base

ARG WORKSPACE_SCOPE=@conduit/user

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

ARG WORKSPACE_SCOPE=@conduit/user

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
# Production image
#
FROM node:18-slim AS production

WORKDIR /app

# Copy built artifacts and dependencies
COPY --from=installer /app .

# Set production environment
ENV NODE_ENV=production

# Expose port (default Express port)
EXPOSE 3000

# Default command - can be overridden for Lambda or local deployment
CMD ["node", "--version"]
