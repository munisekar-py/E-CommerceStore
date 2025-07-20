# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the service port
EXPOSE 3001

# Run the application
CMD ["node", "server.js"]
