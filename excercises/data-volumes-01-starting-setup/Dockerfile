FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

# This is the same than adding "-v /app/node_modules" in the docker run command. This is an anonymous volume
# created to persist the container node_modules folder even if a bind directory is created in the same parent path(/app).
# Since "/app/node_modules" is more specific it will survive. It's commented because the -v param will be used
# directly in the run command to avoid rebuilding the image. It's anonymous because no name was added (NAME:path)

# VOLUME [ "/app/node_modules" ]

CMD [ "npm", "start" ]