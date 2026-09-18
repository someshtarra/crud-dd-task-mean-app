# Use official Node.js 18 LTS Alpine image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install --production

# Copy application source code
COPY . .

# Expose backend API port
EXPOSE 8080

# Define default environment variables
ENV PORT=8080
ENV NODE_ENV=production

# Start Node.js application
CMD ["node", "server.js"]
