# Bower Buildpack

This buildpack will handle installing Bower dependencies for your app.

## How to use it?

Unless you're deploying a very boring application that has no backend stack (Node.js / Play / Rails / Django / whatever), you're going to want to use this buildpack along with your primary buildpack. You do that by using [heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi).

Assuming your application is Node.js, using this buildpack along with the Node.js buildpack is as simple as checking in a `.buildpacks` file into your repository...

    $ cat .buildpacks

    https://bitbucket.org/sday_atlassian/heroku-buildpack-bower.git#master
    https://github.com/heroku/heroku-buildpack-nodejs.git#master

... and then updating your Heroku app to use the heroku-buildpack-multi buildpack.

    $ heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git

## How does it do it?

This buildpack will vendor a private copy of Node.js to run Bower. The Node.js install is fetched from Heroku's CDN, so it takes approx ~1 second to download the tarball and extract it. It caches the install of Bower into your slug to speed up the install process. **It also configures Bower to use your slugs cache for the packages + registry metadata, so subsequent re-deploys are very quick.** Finally, it (temporarily) uses a [forked verison of Bower](https://github.com/neoziro/bower/tree/v1.2.8-patched) that ensures Bower doesn't croak due to crazy [ENOENT / ENOTEMPTY](https://github.com/bower/bower/issues/933) issues during deploy.

## Why use it?

Without this buildpack you have two options:

 * Check your `bower_components/` directory in to the repo.
 * Use the [Node.js](https://github.com/heroku/heroku-buildpack-nodejs) buildpack, add a package.json that depends on bower, and a `postinstall` step that runs bower.

## TODO

 * Bower install is cached a little too aggressively - right now it'll never be updated after the initial install. The buildpack should put the date in a .bower-install file in the cache, and update Bower if that date is older than, say, a week.
