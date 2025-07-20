# Step 1: Build the React app

FROM node:18-alpine AS builder

# Set working directory

WORKDIR /app

# Copy dependencies

COPY package.json package-lock.json* ./

# Install dependencies

RUN npm install

# Copy the rest of the app

COPY . .

# Build the React app

RUN npm run build

# Step 2: Serve the built app using nginx

FROM nginx:stable-alpine

# Remove default nginx static assets

RUN rm -rf /usr/share/nginx/html/*

# Copy built React app from builder

COPY --from=builder /frontend/public/* /usr/share/nginx/html

# Copy custom nginx config (optional)

# COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port

EXPOSE 80

# Start nginx

CMD ["nginx", "-g", "daemon off;"]
