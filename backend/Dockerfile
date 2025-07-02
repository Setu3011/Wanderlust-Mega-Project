# Stage 1: Build
FROM node:21-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install --only=production

COPY . .

# Stage 2: Run
FROM node:21-alpine

WORKDIR /app

COPY --from=builder /app /app

COPY .env.docker .env

EXPOSE 8080

# Use node instead of nodemon for production
CMD ["node", "server.js"]

