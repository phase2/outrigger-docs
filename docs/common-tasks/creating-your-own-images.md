# Creating your own images

## Creating a Dockerfile

The Dockerfile you create for your project should represent the application as it will need to run in production. There 
are many reasons to have a Dockerfile to create an image, but if you don't intend on running containers in production 
you can likely skip this part. 

The most minimal project container will need to have your code in it.

* Start by determining which Docker image you will need to extend. 

* For the general Drupal use cases you will base your project Dockerfile on apache-php:php70.

    * `FROM phase2/apache-php:php70`

* Then copy your code into the correct place for the container. For a PHP/Drupal project, copy the materialized site 
code into the container docroot

    * `COPY ./html /var/www/html/`

* Build the Dockerfile into an image

    * `docker build -t <some-name> .`

* On successful build, test the image by running

    * `docker run -t <some-name>`

Here is the full (simple) Dockerfile

```dockerfile
FROM phase2/apache-php:php70

# Copy in the site
COPY ./html /var/www/html/
```
