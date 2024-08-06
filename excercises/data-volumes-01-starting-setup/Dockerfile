FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

# This is an anonymous Volume. In volume the given path is the path in the container.
VOLUME [ "/app/feedback" ]

CMD [ "node", "server.js" ]