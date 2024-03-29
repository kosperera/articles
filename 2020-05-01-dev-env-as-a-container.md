---
title: Homogenize your dev environments with Docker
subtitle: Creating a dead simple dev environment container
slug: dev-env-as-a-container
# previously published url.
canonical: https://kosalanuwan.github.io
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: developer-tools, future, docker, productivity
cover: https://user-images.githubusercontent.com/958227/233758737-5287107d-d989-4641-81be-5d3aab67309d.jpeg?auto=compress
domain: keepontruckin.hashnode.dev
---

I need to build an eng-practice wiki for Alertbox Inc. (@[alertboxinc](@alertboxinc)) and most of these contents are pretty outdated word docs and slide decks. It's not such a great idea and probably not even fun to begin with since, Alertbox has strict security policies on work PC.

<img class="img-responsive img-comic" alt="Teaching capitalism for the communist mud planning pigs from the Elbonia - Dilbert by Scott Adams" src="https://user-images.githubusercontent.com/958227/227763961-3355c49d-3697-4e2c-b172-2cabf9454491.gif" width="100%">

For the record, not only are there no administrative rights for developers, but there's no permission at all. It's a *Brick*, as in barely anything you can do with it. The truth is, I'm way too lazy to go thru their intense approval process to set up a dev environment. I need to <mark>keep the *Hacktivation Effort* to minimal.</mark>

[Dan Wahlin's Docker for Web Developers course][pluralsight-dan-wahlin-docker-for-web-devs] is open on my work PC and he is showing how to hook the source code into a container. Good idea Mr. Wahlin!

[pluralsight-dan-wahlin-docker-for-web-devs]: https://app.pluralsight.com/library/courses/docker-web-development/table-of-contents

I'm not exactly sure how the wiki is going to fit into this whole thingy, but I like the idea of developing inside a container. It gives the whole thing a very *Homogenize* feel since each teammate's work PC will be more implicit than, say, installing all the prerequisite dev tools on their work PC like we typically do. Easy enuf, except, <mark>I need to bake in development environments as part of source repos.</mark>

[dev-containers-quick-start]: https://docs.docker.com/language/nodejs/develop/

The first thing we're going to need is a markdown parser to generate the site like they do in [Microsoft Docs][ms-docs] sites. And that means [Docsify][docsify-quick-start]. They are basically similar to [Jekyll][jekyll-quick-start], but with no build steps. All I need to do is fork their [docsify-template][docsify-template-repo] repo and we're good to go!

[ms-docs]: https://learn.microsoft.com/en-us/docs/
[docsify-quick-start]: https://docsify.js.org/#/quickstart
[jekyll-quick-start]: https://jekyllrb.com/docs/
[docsify-template-repo]: https://github.com/docsifyjs/docsify-template

The other thing we're going to need is a dead simple [node container][docker-images-nodejs-official] to run docsify locally, so a little `docker run` magic is all that's necessary to [sync the repo contents][docker-cli-map-source-dir] and spawn the container. Then I can [run `npx serve` to spin up an HTTP server][npm-serve] to preview on localhost and docsify will parse the markdown to generate the site on the fly for me. Kids' stuff!

[docker-images-nodejs-official]: https://hub.docker.com/_/node/tags?page=1&name=bullseye
[docker-cli-map-source-dir]: https://docs.docker.com/engine/reference/commandline/run/#volume
[npm-serve]: https://www.npmjs.com/package/serve

```bash
docker run --name stoic_goldstine \
           --rm \                    # clean up
           -w /workspaces \          # working directory
           -v "$(pwd):/workspaces"   # sync immediate file changes to container
           -p 3000:3000 \            # expose port 3000
           node:lts-bullseye \       # node image
           npx serve . --listen 3000 # run the http server in port 3000
```

Unfortunately, docker runs in the foreground and requires another Terminal to stop to and remove the container. It would be impossible to run in [detached `-d` mode][docker-cli-detached-mode] without giving away docker [clean up `--rm` option][docker-cli-clean-up] completely. And you do want docker to automatically clean up when the container exit. It's time to switch to [Docker Dev][docker-dev-overview].

[docker-dev-overview]: https://docs.docker.com/desktop/dev-environments/
[docker-cli-detached-mode]: https://docs.docker.com/engine/reference/run/#detached--d
[docker-cli-clean-up]: https://docs.docker.com/engine/reference/run/#clean-up---rm

<img class="img-responsive img-comic" alt="Docker dev environment in action - Docker Desktop" src="https://user-images.githubusercontent.com/958227/227768140-b623c0ee-6f45-4d32-a641-88372e44e873.png" width="100%">

I prefer `docker dev` over `docker`. They're technically the same as `docker compose`, except `docker dev` is specifically for dev environments. It also requires [a compose YAML file to configure the container][docker-compose-create-yaml], so you don't need to memorize which image to pull, what ports to expose, the working directory to map, args to override, et al. All I need to do is break out the same `docker` command that I use to spawn the container in detached mode to [create a `compose-dev.yaml` file][compose-dev-yaml] and we're all set.

[docker-compose-create-yaml]: https://docs.docker.com/get-started/08_using_compose/#create-the-compose-file
[compose-dev-yaml]: https://docs.docker.com/desktop/dev-environments/set-up/

```yaml
# ./compose-dev.yml
services:
  node:
    image: node:lts-bullseye
    container_name: stoic_goldstine
    ports:
      - 3000:3000
    working_dir: /workspaces
    volumes:
      - ./:/workspaces
    command: /bin/sh -c "npx serve . --listen 3000"
```

Then I can use `docker dev` commands to spawn the dev environment and docker will do the rest for me. Excellent!

```bash
#!/bin/zsh
# Spin up the dev environment
docker dev create $(pwd)
# Stop the dev environment
docker dev stop devcontainers-try-docsify-hungry_elion
```

> **Note**
>
> [Docker Dev CLI][docker-dev-cli] is handy if you would want to stick to the Terminal. Use [Docker Desktop][docker-desktop-download] or the [Browser Extension][docker-dev-browser-ext], to create and [launch dev environments][docker-dev-launch-dev-envs] otherwise.

[docker-dev-cli]: https://docs.docker.com/desktop/dev-environments/dev-cli/
[docker-desktop-download]: https://www.docker.com/products/docker-desktop/
[docker-dev-browser-ext]: https://github.com/docker/dev-envs-extension
[docker-dev-launch-dev-envs]: https://docs.docker.com/desktop/dev-environments/create-dev-env/



[I use Typora for markdown][hello-world] but we're going to have to change the default theme. They require intellisense and syntax highlighting to work with HTML/CSS and JavaScript. I can use [VS Code][vscode-download] with a few [extensions](./.vscode/extensions.json) but there are more problems.

<img class="img-responsive img-comic" alt="They write high-quality code for six cents a day - Dilbert by Scott Adams" src="https://user-images.githubusercontent.com/958227/227764203-bef9e1fd-39d6-40f9-a62e-b865fb55477c.gif" width="100%">

Not only is there no language recognition, but no intellisense at all. I'm going to go ahead and say that they don't have access to the node runtime inside the container, so they have no way of knowing what syntax highlighting and intellisense to show. And you do want to them to work. Time to install the [Remote - Containers][vscode-remote-containers-extension] extnesion and `Attach to Running Container`. Then I can install the required extensions for node and JavaScript. It's slightly obnoxious that VS Code doesn't install recommended extensions from the `.vscode/extensions.json` automatically, so I'm going to have to install `recommendations` manually.

[hello-world]: https://keepontruckin.hashnode.dev/hello-world
[vscode-download]: https://code.visualstudio.com/
[vscode-remote-containers-extension]: https://aka.ms/vscode-remote/download/containers

Unfortunately, there's no simpler way of converting word docs and slide decks into markdown so I'm going to have to copy all the contents from the individual directories that are in. And there's no way I'm going to go to 100 pages and slides to create markdown content one at a time, so it's definitely necessary to push this initial content to a git repo and let the teammates contribute.



/KP



> **PS:** For the complete work, see [my devcontainers-try-docsify repo on GitHub][gh-repo].

[gh-repo]: https://github.com/alertbox/devcontainers-try-docsify/tree/devenvs-try-docsify#readme
