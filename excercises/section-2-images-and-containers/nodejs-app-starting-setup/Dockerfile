FROM node

WORKDIR /app

# Copy separately the package.json and doing the installation before copying all app files
# is an optimization to avoid unnecessary rerun of npm install when changes in the app.
COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node", "server.js"]