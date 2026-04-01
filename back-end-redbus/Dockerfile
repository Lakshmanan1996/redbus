# ---------- BASE IMAGE ----------
FROM node:20-alpine

# Create app directory
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy remaining source code
COPY . .

# Expose backend port (check app.js, usually 5000 or 3000)
EXPOSE 3000

# Start the app
CMD ["node", "app.js"]
