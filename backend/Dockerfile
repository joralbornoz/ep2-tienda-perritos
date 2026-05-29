FROM node:26-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --ignore-scripts --omit=dev

COPY . .

EXPOSE 3001

CMD ["node", "server.js"]