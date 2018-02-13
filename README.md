# Blue-green deployments for Cloud Foundry based Bluemix apps

Allows zero-downtime deployments of applications within Bluemix, with no additional setup needed.

This repo is heavily based off of [cf-blue-green](https://github.com/18F/cf-blue-green)

# When to use bx-blue-green instead of cf-blue-green

This project uses the bluemix command line directly allowing it to leverage some of the new features added to bluemix.

The Bluemic CLI adds extra functionality to Cloud Foundry command line, if you are deploying onto Bluemix, it is best to migrate over to the Bluemix CLI as it will contain the latest functionality for Bluemix related deployments.

One notable feature that is now part of the Bluemix CLI is the ability to create an API key in order to avoid having to use a One-time login token. 

For steps on creating a Bluemix API key [click here](#bluemix-api-key)

## Bluemix API key

To generate your Bluemix API key follow these steps:

![demo](docs/bluemix-api-key.gif)

alternatively you can run 

`bluemix iam api-key-create <key-name> -d <description> -f <output-file>`

Once the key is created, you may create an environment variable with the key `BLUEMIX_API_KEY` and when running `bx login` it will check for that variable first and if it exists, it will log you in.

## Usage

1. [Install the `bx` CLI](https://github.com/cloudfoundry/cli/releases) **v0.6.0+**.
1. Run `npm install -g https://github.com/andresfvilla/bx-blue-green`.
    * See [Notes](#manual-installation) below for non-Node installation.
1. Run `bx-blue-green <appname>` (instead of `bx cf push`) from your application directory to deploy.

This creates a copy of your already-running application, and safely switches traffic over to it. It's recommended that you try this script on a non-production application environment first, just to ensure that everything is switched over properly.

## Notes

### Manual installation

The script is distributed via NPM, but doesn't actually require Node.js beyond that. If you don't want to install Node, simply:

1. Download [the script](bin/bx-blue-green).
1. Run `chmod a+x bx-blue-green`.
1. Move the file somewhere in your PATH.

### Using with Travis

Travis supports [continuous deployment](http://docs.travis-ci.com/user/deployment/), which will automatically deploy your application after its tests pass on a specified branch. To use `bx-blue-green` with Travis, you need to use a [script provider](http://docs.travis-ci.com/user/deployment/script/) instead of the default Cloud Foundry provider. Your Cloud Foundry settings are read from environment variables.

Set up continuous deployment with the following settings in your `.travis.yml` file:

To add an encrypted Bluemix API key use `travis encrypt BLUEMIX_API_KEY=<api-key> --add`

```yml
sudo: true
env:
  global:
  - BX_APP=[app name]
  - BX_API=[API endpoint]
  - BX_ORGANIZATION=[organization]
  - BX_SPACE=[space]
  - BX_SLEEP=[# seconds to wait before swapping BLUE with GREEN]
  - $BX_IGNORE_DEFAULT_ROUTE=[true/false Will ignore the orgs default route when deploying new app]
  - secure: [BLUEMIX_API_KEY=[encrypted with Travis](http://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables)]
before_deploy: npm install -g https://github.com/andresfvilla/bx-blue-green
deploy:
  provider: script
  script: bx-blue-green-travis
  on:
    branch: [git branch you want to deploy]
```

`BX_SLEEP` - defines the number of seconds to sleep before replacing BLUE with GREEN. Useful when the new application requires some time to start.

`B_DOMAIN` - Use this to specify a specific domain for where your green app will be stood up ( Usually equal to your default route )

`BX_IGNORE_DEFAULT_ROUTE` - will force the green app to use the domain specified in `B_DOMAIN`. If your green app is being deployed to a route you specifically do not want, set this variable to true. If this is set, then `B_DOMAIN` is required

### Manifests

`bx-blue-green` creates a temporary manifest from your live application, meaning that it ignores the `manifest.yml` in your directory, if you have one. To deploy any changes to your manifest, use `bx cf push` directly.

## Multiple domains

The script fails on apps with multiple domains, because the domains in the manifest are in the form of a list:

```yml
domain:
  - 18f.gov
  - digitalgov.gov
```

To work around this, use the env var `B_DOMAIN` for the domain you'd like the B instance to use.


## Resources

More information about blue-green deployment, all of which this script drew from.

* http://www.cloudfoundry.rocks/blue-green-deployment-with-cloudfoundry/
* http://martinfowler.com/bliki/BlueGreenDeployment.html
* http://docs.pivotal.io/pivotalcf/devguide/deploy-apps/blue-green.html
* https://github.com/dlapiduz/step-cloud-foundry-deploy/blob/master/run.sh
