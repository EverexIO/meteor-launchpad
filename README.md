[![Circle CI](https://circleci.com/gh/jshimko/meteor-launchpad/tree/master.svg?style=svg)](https://circleci.com/gh/jshimko/meteor-launchpad/tree/master)
# Meteor Launchpad - Base Docker Image for Meteor Apps

### Build

Add the following to a `Dockerfile` in the root of your app:

```Dockerfile
FROM jshimko/meteor-launchpad:latest
```

Then you can build the image with:

```sh
docker build -t yourname/app .
```

### Run

Now you can run your container with the following command...
(note that the app listens on port 3000 because it is run by a non-root user for [security reasons](https://github.com/nodejs/docker-node/issues/1) and [non-root users can't run processes on port 80](http://stackoverflow.com/questions/16573668/best-practices-when-running-node-js-with-port-80-ubuntu-linode))

```sh
docker run -d \
  -e ROOT_URL=http://example.com \
  -e MONGO_URL=mongodb://url \
  -e MONGO_OPLOG_URL=mongodb://oplog_url \
  -e MAIL_URL=smtp://mail_url.com \
  -p 80:3000 \
  yourname/app
```

### Custom Build Options

Meteor Launchpad supports a few custom build options by using a config file in the root of your app.  The currently supported options are to add PhantomJS or MongoDB to your build.  To install either of them, create a `launchpad.conf` in the root of your app and add either of the following values.

```sh
# launchpad.conf

INSTALL_PHANTOMJS=true
INSTALL_MONGO=true
INSTALL_GRAPHICSMAGICK=true
```

If you choose to install Mongo, you can use it by _not_ supplying a `MONGO_URL` when you run your app container.  The startup script will then start Mongo and tell your app to use it.  If you _do_ supply a `MONGO_URL`, Mongo will not be started inside the container and the external database will be used instead.

Note that having Mongo in the same container as your app is just for convenience while testing/developing.  In production, you should use a separate Mongo deployment or at least a separate Mongo container.


## Development Builds

You can optionally avoid downloading Meteor every time when building regularly in development.  Add the following to your Dockerfile instead...

```Dockerfile
FROM jshimko/meteor-launchpad:devbuild
```

This isn't recommended for your final production build because it creates a much larger image, but it's a bit of a time saver when you're building often in development.  The first build you run will download/install Meteor and then every subsequent build will be able to skip that step and just build the app.

## Docker Compose

Add a `docker-compose.yml` to the root of your project with the following content and edit the app image name to match your build name.  Everything else should work as-is.

```yaml
# docker-compose.yml

app:
  image: yourname/app
  ports:
    - "80:3000"
  links:
    - mongo
  environment:
    - ROOT_URL=http://example.com
    - MONGO_URL=mongodb://mongo:27017/meteor

mongo:
  image: mongo:latest --storageEngine=wiredTiger
```

And then start the app and database containers with...

```sh
docker-compose up -d
```
