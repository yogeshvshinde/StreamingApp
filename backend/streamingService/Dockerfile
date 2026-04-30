FROM node:18-alpine

WORKDIR /app/streamingService

COPY package*.json ./
RUN npm install --production

COPY . .
ENV NODE_ENV=production
EXPOSE 3002

CMD ["npm", "run", "start"]
