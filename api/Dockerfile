
# Use Node.js 20 as the base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package files from project root
COPY package*.json ./

# Install production dependencies
RUN npm install --omit=dev

# Copy backend source code
COPY api ./api

# Backend listens on port 5000
EXPOSE 5000

# Start Express backend
CMD ["node", "api/index.js"]
