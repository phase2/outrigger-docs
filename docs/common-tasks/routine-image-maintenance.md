# Routine Image Maintenance

Restart rig on a regular basis to minimize the risk of filesystem or other performance problems with your containers.

## Docker Image Updates

Periodically update your Docker Images to ensure you have the correct configuration. This should be done in coordination
with members of your project team so you are all working from the same version of image. Ideally all images in compose 
files are working off of tags to ensure consistency.  Images should be pulled frequently though especially for images
that are tagged with a language version, like *php70*. Updates may go into those images and keep the same tag, so pulling
regularly (and coordinating) will ensure you are getting state of the art.

If your project uses "loosely versioned" Docker images (such as specifying *latest* as an image version tag or no tag
at all which will default to *latest*) your local update schedule should be coordinated with updates to other environments
and team members.

Here is how to update your images

```bash
# Stop your containers in case they are running.
docker-compose stop

# Pull the latest changes for all images referenced in docker-compose.yml
docker-compose pull

# Pull the latest changes for all images referenced in build.yml.
docker-compose -f build.yml pull

# Remove your old containers to make sure you're using the latest images.
docker-compose rm

# Re-create your containers based on the freshly updated images.
docker-compose up -d
```

## Updates for a Specified Image

```bash
docker pull phase2/build:php70
```

If either form of `pull` command fails, try re-running with the `--no-cache` option.
