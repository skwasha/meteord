[![Circle CI](https://circleci.com/gh/meteorhacks/meteord/tree/master.svg?style=svg)](https://circleci.com/gh/meteorhacks/meteord/tree/master)
## MeteorD - Docker Runtime for Meteor Apps 

There are two main ways you can use Docker with Meteor apps. They are:

1. Build a Docker image for your app
2. Running a Meteor bundle with Docker

**MeteorD supports these two ways. Let's see how to use MeteorD**

### 1. Build a Docker image for your app

With this method, your app will be converted into a Docker image. Then you can simply run that image.  

For that, you can use `meteorhacks/meteord` as your base image. Magically, that's only you've to do. Here's how to do it.

Add following `Dockerfile` into the root of your app:

~~~shell
FROM meteorhacks/meteord:onbuild
~~~

Then you can build the docker image with:

~~~shell
docker build -t yourname/app .
~~~

Then you can run your meteor image with

~~~shell
docker run -d \
    -e ROOT_URL=http://yourapp.com \
    -e MONGO_URL=mongodb://url \
    -e MONGO_OPLOG_URL=mongodb://oplog_url \
    -p 8080:80 \
    yourname/app 
~~~

#### Stop downloading Meteor each and every time (mostly in development)

So, with the above method, MeteorD will download and install Meteor each and every time. That's bad specially in development. So, we've a solution for that. Simply use `meteorhacks/meteord:devbuild` as your base image.

> WARNING: Don't use `meteorhacks/meteord:devbuild` for your final build. If you used it, your image will carry the Meteor distribution as well. As a result of that, you'll end up with an image with ~700 MB.

Then you can access your app from the port 8080 of the host system.

### 2. Running a Meteor bundle with Docker

For this you can directly use the MeteorD to run your meteor bundle. MeteorD can accept your bundle either from a local mount or from the web. Let's see:

#### 2.1 From a Local Mount

~~~shell
docker run -d \
    -e ROOT_URL=http://yourapp.com \
    -e MONGO_URL=mongodb://url \
    -e MONGO_OPLOG_URL=mongodb://oplog_url \
    -v /mybundle_dir:/bundle \
    -p 8080:80 \
    meteorhacks/meteord:base
~~~

With this method, MeteorD looks for the tarball version of the meteor bundle. So, you should build the meteor bundle for `os.linux.x86_64` and put it inside the `/bundle` volume. This is how you can build a meteor bundle.

~~~shell
meteor build --architecture=os.linux.x86_64 ./
~~~

#### 2.1 From the Web

You can also simply give URL of the tarball with `BUNDLE_URL` environment variable. Then MeteorD will fetch the bundle and run it. This is how to do it:

~~~shell
docker run -d \
    -e ROOT_URL=http://yourapp.com \
    -e MONGO_URL=mongodb://url \
    -e MONGO_OPLOG_URL=mongodb://oplog_url \
    -e BUNDLE_URL=http://mybundle_url_at_s3.tar.gz \
    -p 8080:80 \
    meteorhacks/meteord:base
~~~


### Rebuilding Binary Modules

Sometimes, you need to rebuild binary npm modules. If so, expose `REBULD_NPM_MODULES` environment variable. It will take couple of seconds to complete the rebuilding process.

~~~shell
docker run -d \
    -e ROOT_URL=http://yourapp.com \
    -e MONGO_URL=mongodb://url \
    -e MONGO_OPLOG_URL=mongodb://oplog_url \
    -e BUNDLE_URL=http://mybundle_url_at_s3.tar.gz \
    -e REBULD_NPM_MODULES=1 \
    -p 8080:80 \
    meteorhacks/meteord:binbuild
~~~
