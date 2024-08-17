# Deploying docker containers

## Manual deployment

### Resources links

- [AWS the basics](https://academind.com/tutorials/aws-the-basics)
- [What is Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)

### Install docker in remote instance (AWS)

1. `sudo yum update`
2. `sudo yum -y install docker`
3. `sudo service docker start`
4. `sudo usermod -a -G docker ec2-user`
5. `sudo chmod 666 /var/run/docker.sock`
6. `docker version`

### Deploying and running image in instance

The best way to do that is to deploy a built image.

1. build your image in your local host.
2. Create your public repository in the registry(E.g Dockerhub)
3. Retag your image to the repository name (`docker tag local-image-name username/repository-name`)
4. Push you image to the repository
5. Pull or run directly the container in the instance. (`docker run -d --rm -p 80:80 username/node-deployment-example-1`)

### Allow remote connection to instance (Security groups inbound rules)

1. Check the Security group associated to you instance and edit it's inbound rules.
2. Add a rule of type `http` and source `anywhere`

## Managed Remote Machines (AWS Elastic Container Services - ECS)

Pending lectures: from 141 to 163.