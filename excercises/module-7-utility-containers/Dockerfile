FROM node:14-alpine

# In Linux systems the default Docker runs as root, so created files like the package.json will have root permissions.
# In systems where uid:gid is also 1000:1000, this happens to map nicely to the "node" user defined within the Official
# Node Docker Image. So, in case of using the Official Node Docker Container, all you need to do is make sure you
# specify that you want the container to run as a non-Root user that they make available.
USER node

WORKDIR /app

ENTRYPOINT [ "npm" ]