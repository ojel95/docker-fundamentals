FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 8000

ENV TASKS_FOLDER=tasks

RUN mkdir ${TASKS_FOLDER}

CMD [ "node", "tasks-app.js" ]