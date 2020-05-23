FROM node:10-slim as prod
ENV NODE_ENV=production
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=prod
COPY . .
EXPOSE 3000
CMD ["node", "./bin/www"]

FROM prod as dev
ENV NODE_ENV=development
RUN npm install
CMD ["./node_modules/nodemon/bin/nodemon.js", "./bin/www", "--inspect=0.0.0.0:9229"]

FROM dev as test
ENV NODE_ENV=development
CMD ["npm", "test"]