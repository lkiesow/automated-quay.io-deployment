Quay.io Deployment Tools
========================

[Quay.io](https://quay.io) is a nice tool for automatically building and
hosting your Docker containers. It supports [Github based build triggers
](https://coreos.com/quay-enterprise/docs/latest/github-build.html) to
automatically build containers every time you push to your repository which is
really nice.

But it does not support any form of automatic rebuilds to ensure that base
images get updated. This essentially means that unless you regularly push to
your Dockerfile repository, your base images will never get updated.

This repository contains a very minimal script to support a custom trigger
using [Travis CI](https://travis-ci.com). Since Travis supports not only build
triggers but also cronjobs, this makes regular builds easy.


Setting up Quay.io
------------------

*Note that this will work for public repositories only.*

Go to the “Builds” section in your repository on quay.io and select “Create
Build Trigger” and “Custom Git Repository Push”. In the following dialog, use
the HTTPS variant for your Github repository URL. That makes it completely
unnecessary to deal with any SSH keys later on.

After it has been successfully created, you will get a Webhook Endpoint URL
which you will need for Travis. The URL should look somewhat like this:

    https://$token:...@quay.io/webhooks/push/trigger/...


Setting up Travis CI
--------------------

Start by enabling Travis CI for your repository as usual. Then go into your
repository settings on Travis and add an environment variable called
`QUAY_WEBHOOK_URL` containing the Quay Webhook Endpoint URL you received
earlier.

Then add a `.travis.yml` build configuration file like this to your repository:

```yaml
sudo: false
script:
  - curl -s https://raw.githubusercontent.com/lkiesow/automated-quay.io-deployment/master/deploy-on-quay | sh
```


This will let you build fail if the deployment to quay did not work. If you
want this to fail silently instead (e.g. to not disrupt development), you can
use [Travis script deployment
](https://docs.travis-ci.com/user/deployment/script) instead:

```yaml
sudo: false
install:
  - curl -O https://raw.githubusercontent.com/lkiesow/automated-quay.io-deployment/master/deploy-on-quay
script:
  - true
deploy:
  provider: script
  script: sh deploy-on-quay
  on:
    branch: master
```

Additionally, you can set the Quay.io default branch by setting the
`QUAY_DEFAULT_BRANCH` environment variable. This is set to `master` by default.
The default branch is used by Quay.io to determine if a container that is built
gets tagged with the `latest` tag.


### A Note About Security

Instead of loading the script from a foreign repository and just execute it it
is obviously possible to directly include this script in the Dockerfile
repository instead.
