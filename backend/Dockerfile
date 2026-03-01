# Backend Dockerfile
FROM node:20-slim

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

# Easypanel uses PORT 3000 by default but my server uses 3001
# I will match it to 3001
EXPOSE 3000

CMD ["node", "server.js"]
