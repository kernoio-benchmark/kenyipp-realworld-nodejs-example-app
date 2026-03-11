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

# Copy necessary files to the installer stage
COPY .gitignore .gitignore
COPY --from=base /app/out/json/ .
COPY --from=base /app/out/yarn.lock ./yarn.lock

# Install dependencies
RUN yarn install --frozen-lockfile

# Build the project using the variable scope
COPY --from=base /app/out/full/ .
RUN yarn global add turbo
RUN turbo run build --filter=$WORKSPACE_SCOPE...

#
# Runner image - production runtime
#
FROM node:18-slim AS runner

ARG WORKSPACE_SCOPE=@conduit/article

# Set working directory
WORKDIR /app

# Set NODE_ENV to production
ENV NODE_ENV=production

# Copy the entire built workspace with all dependencies
# This includes all internal packages and node_modules
COPY --from=installer /app .

# Expose the application port (default Express port)
EXPOSE 3000

# Set the default command to run the application
CMD ["node", "apps/article/index.js"]
