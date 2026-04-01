# =========================================================
#  React (Frontend – Production Build)
# =========================================================

# Build stage
FROM node:20 AS build

WORKDIR /app
COPY front-end-redbus/package*.json ./
RUN npm install
COPY front-end-redbus/ .
RUN npm run build

# Runtime stage
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
