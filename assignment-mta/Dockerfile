FROM node:8
ENV CHARCOAL_FACTOR=0.1
RUN apt-get update && apt-get install -y graphicsmagick
COPY package*.json ./
RUN npm install && npm cache clean --force
COPY . .
CMD ["node", "./index.js"]