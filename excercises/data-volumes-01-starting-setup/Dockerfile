FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

# This defines a variable with a default value that can be set during build time.
ARG DEFAULT_PORT=80

# This defines env variable to be used by the container/app in runtime.
ENV PORT $DEFAULT_PORT

#Here the harcoded port number is changed to use the env variable defined before.
EXPOSE $PORT

# This is the same than adding "-v /app/node_modules" in the docker run command. This is an anonymous volume
# created to persist the container node_modules folder even if a bind directory is created in the same parent path(/app).
# Since "/app/node_modules" is more specific it will survive. It's commented because the -v param will be used
# directly in the run command to avoid rebuilding the image. It's anonymous because no name was added (NAME:path)

# VOLUME [ "/app/node_modules" ]

CMD [ "npm", "start" ]