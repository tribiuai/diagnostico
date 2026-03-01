# Frontend Dockerfile
FROM node:20-slim as build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Stage 2: Serve the static files
FROM node:20-slim

WORKDIR /app
RUN npm install -g serve

COPY --from=build /app/dist ./dist

# Vite preview/serve default is often 3000 or 4173
# I will set it to 3000 for Easypanel
EXPOSE 3000

CMD ["serve", "-s", "dist", "-l", "3000"]
