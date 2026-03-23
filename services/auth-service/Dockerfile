FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS development
RUN npm install
COPY . .
EXPOSE 3001
CMD ["npm", "run", "dev"]

FROM base AS production
RUN npm ci --only=production
COPY . .
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 3001
CMD ["node", "src/app.js"]
