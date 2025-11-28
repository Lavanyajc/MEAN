# Stage 1: Build Angular application
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build --prod

# Stage 2: Serve Angular using Nginx
FROM nginx:alpine

# Copy built Angular files from Stage 1
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html

# Expose internal Nginx port
EXPOSE 80

# Nginx starts automatically (no CMD needed)
