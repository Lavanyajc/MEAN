# Use lightweight Node.js image
FROM node:18-alpine

# Create working directory inside container
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# Copy the rest of the backend source code
COPY . .

# Backend will run on port 3000 inside the container
ENV PORT=3000
EXPOSE 3000

# Start the backend server
CMD ["node", "server.js"]
