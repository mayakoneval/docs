import withDoc from '../../../lib/with-doc'

import Caption from '../../../components/text/caption'
import { TerminalInput } from '../../../components/text/terminal'

export const meta = {
  title: 'Build-Time Environment Variables and Secrets',
  description: 'Creating and using environment variables and creating and using secret environment variables during the build stage of your Now deployments',
  date: '19 July 2018',
  editUrl: 'pages/docs/features/build-env-and-secrets.md',
  image: IMAGE_ASSETS_URL + '/docs/build-env/twitter-card.png'
}

> If you're looking for information covering Environment Variables and Secrets to use within your app, head over to our guide for [Run-Time Env and Secrets](/docs/features/env-and-secrets)

There are multiple stages to deploying your app, for example installing dependencies. For Now to recognize that your app needs a build step, your app will need a `Dockerfile`.

Having a `Dockerfile` is a brilliant way to configure the process of your app being built so that Now will be able to build your app and then deploy using your configuration.

During your build, you may need information that you do not want to hard code into your app or want to keep in a configuration for the app. In this case, Now supports a method that allows you to give an environment variable or secret for your deployment to use where you need that information.

## Step 1: Setting up Build Environment Variables and Secrets
To use your custom environment variables or secrets in the build process on Now, there are two methods.

### The `now.json` method
You can provide your Now deployment with environment variables directly from within your `now.json` configuration file. To do so for the build step, you will need to place this information within the `build.env` property. For example:

```
{
  "build": {
    "env": {
      "NODE_ENV": "production"
    }
  }
}
```
<Caption>Listing the `NODE_ENV` build environment variable with the value of `production` within a `now.json` configuration file and within the `build.env` property.</Caption>

### The Now CLI Method
You can use the built-in `--build-env` parameter to pass your environment variables to the build stage of your Now deployment. For example, the following command will give the `NODE_ENV` environment variable the value of `production`.

<TerminalInput>now --build-env NODE_ENV=production</TerminalInput>
<Caption>Setting the `NODE_ENV` build environment variables to have a value of `production` with the Now CLI</Caption>

### Using Secrets
Secrets are a way to keep information from being displayed in your configuration or in your code. [Now will store secrets associated to your account or team](/docs/features/env-and-secrets#securing-env-variables-using-secrets), ready for usage with environment variables.

To use secrets with Now in the build stage, we first need to add our secret with the `now secret` command in the Now CLI.

<TerminalInput>now secret add npm-token NPM_TOKEN_VALUE</TerminalInput>
<Caption>Adding the `npm-token` secret to your account or team, with the example of a value</Caption>

We can now use this secret at build-time by passing it as an environment variable, using both methods above but by using an `@` symbol before the value.

For example, in a `now.json` configuration, it would look like this:

```
{
  "build": {
    "env": {
      "NPM_TOKEN": "@npm-token"
    }
  }
}
```
<Caption>Using our `npm-token` that we added to our account or team before with an environment variable within a `now.json` configuration.</Caption>

You can also use secrets in the CLI using the same method of using the `@` symbol before the value

Read more about [secrets within Now](/docs/features/env-and-secrets#securing-env-variables-using-secrets) for more examples and information.

## Step 2: Using Build Environment Variables and Secrets
When we have set up our environment variables using one of the methods above, with secrets or not, we are now ready to use them within the build step.

By creating a `Dockerfile` we can let Now know that we want a build step for our app. Within that `Dockerfile`, we can use our environment variables that we setup in previous steps (including those with values associated with secrets) using the [`ARG` instruction](https://docs.docker.com/engine/reference/builder/#arg).

For example, if we want to use the `NPM_TOKEN` environment variable from our [last step](#using-secrets) we can use the following in a `Dockerfile`:
```
ARG NPM_TOKEN
```
<Caption>Creating an ARG variable with the value of an earlier created environment variable</Caption>

The `Dockerfile` and Now will associate the value of that ARG with the `NPM_TOKEN` environment variable. We can then subsequently use this value by referencing the ARG with the same name with a `$` prefixing it.

```
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > ~/.npmrc
```
<Caption>Using an environment variable and ARG in conjunction to output a valid `.npmrc` for private npm</Caption>

> Note that by default your deployment will be detected as a Docker deployment when you have a Dockerfile. If you're using the Dockerfile to build a static app, make sure you mark your app as [`"type": "static"` within your now.json configuration](docs/features/static-builds#step-3:-configuring-now-for-static-deployments).

## Example
Now that we've gone through the steps and methods of using environment variables and secrets in the build step, let's take a look at an example.

### Using Private npm packages
To use private npm packages, npm requires that we provide an auth token. We don't always want to hardcode this auth token, so we can use environment variables and secrets to manage this.

To start, let's get a new token. npm will give us a new token using the command `npm token create --read-only` but we probably don't want to share that token around. To protect the token from getting out for anyone else to use, we can use the `now secret` command.

<TerminalInput>now secret add npm-token NPM_TOKEN_VALUE</TerminalInput>
<Caption>Adding the `npm-token` secret to our account or team.</Caption>

Once our secret is added, we can use that secret in our `now.json` configuration.
```
{
  "build": {
    "env": {
      "NPM_TOKEN": "@npm-token"
    }
  }
}
```
<Caption>Note that you can also use the [Now CLI method](#the-now-cli-method) for this also.</Caption>

> You can extend your `now.json` configuration beyond just `build.env`. [Read more about configuring deployments](/docs/features/configuration).

With our environment variable `NPM_TOKEN` set to the value of our secret contained npm auth token, we can go on to use it.

For this example, we are going to install dependencies for a [static app build](/docs/features/static-builds). Within our `Dockerfile`, we will use an [`ARG` instruction](https://docs.docker.com/engine/reference/builder/#arg) that relates to the recently created `NPM_TOKEN` environment variable.

```
FROM mhart/alpine-node

# Retrieve and relate to our environment variable
ARG NPM_TOKEN

# Print into `.npmrc` with a string using our `NPM_TOKEN` ARG
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > ~/.npmrc

WORKDIR /usr/src

# Install dependencies
COPY package.json yarn.lock ./
RUN yarn
COPY . .

RUN yarn build &&
  yarn export -o /public
```
<Caption>Note that the usage of the ARG variable starts with a `$`. This is necessary for Docker to recognize this as an [ARG variable](https://docs.docker.com/engine/reference/builder/#arg).</Caption>

With the `Dockerfile` complete, we can now go on to deploying with Now:

<TerminalInput>now</TerminalInput>

Through the deployment lifecycle, our build will progress with our configuration including that of our environment variables and secrets!

Feel free to try it out yourself by using our [starter repository](https://github.com/zeit/now-build-env-starter-static).

export default withDoc({...meta})(({children}) => <>{children}</>)
