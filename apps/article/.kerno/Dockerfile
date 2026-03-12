#
# Base image for the server
#
FROM node:18 AS base

ARG WORKSPACE_SCOPE=@conduit/article

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

ARG WORKSPACE_SCOPE=@conduit/article

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
# Runtime image
# Production-ready minimal image
#
FROM node:18-slim AS runtime

ARG WORKSPACE_SCOPE=@conduit/article

# Set working directory
WORKDIR /app

# Set NODE_ENV to production
ENV NODE_ENV=production

# Copy only production dependencies and built artifacts
COPY --from=installer /app/package.json ./package.json
COPY --from=installer /app/yarn.lock ./yarn.lock
COPY --from=installer /app/node_modules ./node_modules
COPY --from=installer /app/packages ./packages
COPY --from=installer /app/apps/article ./apps/article

# Expose the port (default Express port)
EXPOSE 3000

# Start the application from the article app directory
CMD ["node", "apps/article/index.js"]
