# Module 8 Notes

## Setup a laravel project using composer container.

After creating the server, php, mysql and composer services/containers in the docker-compose file, run the following command to create
a laravel project generating the files in the src directory since it is binded. Check the docker-compose and composer.dockerfile files
for more reference.

```
docker-compose run --rm composer create-project --prefer-dist laravel/laravel:^8.0 .
```

## Run services

1. Open src/.env in your editor and change the configuration lines for the database connection as follows

DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret

2. Run the project, `docker-compose up -d --build server` and access it on http://localhost:8000
