FROM node:14.17.3-alpine3.14

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]

