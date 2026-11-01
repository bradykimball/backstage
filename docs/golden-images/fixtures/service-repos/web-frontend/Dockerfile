# Expect: no relation. Docker Hub base, not a golden image.
FROM node:22-alpine
WORKDIR /app
COPY . .
RUN npm ci && npm run build
CMD ["node", "server.js"]
