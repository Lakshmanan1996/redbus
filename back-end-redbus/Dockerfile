# =========================================================
#  Node.js – Multi Stage (Production)
# =========================================================

# Build stage
FROM node:20 AS build

WORKDIR /app
COPY back-end-redbus/package*.json ./
RUN npm install
COPY back-end-redbus/ .


# Runtime stage
FROM node:20-alpine

WORKDIR /app
COPY --from=build /app ./

EXPOSE 3000

CMD ["node", "app.js"]
