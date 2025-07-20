# Base image
FROM node:18

# Set working directory
WORKDIR /

# Copy and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the service port
EXPOSE 3004

# Run the application
CMD ["node", "server.js"]
