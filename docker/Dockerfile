FROM node:lts-alpine

WORKDIR '/usr/app'

COPY ../app/package.json . 
RUN npm install

COPY . .

CMD [ "npm", "run", "start" ]
