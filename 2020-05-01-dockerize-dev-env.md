---
title: Dockerize the dev environment
subtitle: Bake in dev tools, runtimes, and dev dependencies as part of source code
slug: docker-as-a-dev-env
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: developer-tools, future, docker, productivity
cover: https://user-images.githubusercontent.com/958227/226838909-346556cc-13f5-4852-b668-2c697a4af748.jpeg?auto=compress
domain: keepontruckin.hashnode.dev
---

I'm a little procrastinative, not gonna lie. The Alertbox Inc. (@[alertboxinc](@alertboxinc)) eng-practice channel is open on my work PC and most of these word docs and slide decks are pretty outdated. I almost want to replace them with a wiki and have teammates contribute to content as they do in [InnerSource][innersource-about].

[innersource-about]: https://resources.github.com/innersource/fundamentals/

<img src="https://user-images.githubusercontent.com/958227/226828868-191df905-2ed8-4ca5-8517-8f061f8df762.gif" alt="Dilbert comic on 2020-01-02 - Fighting fire with fire" width="100%" align="center">

It's not such a great idea and probably not even pragmatic, since it would require a markdown parser like [Jekyll][jekyll-install] or [Docsify][docsify-install] dev environment, and I'm way too lazy to go thru their intense approval process to install them. For the record, Alertbox has strict policies on work PC. Not only are there no administrative rights to install dev tools, but there's no permission at all. It's a brick, as in barely anything you can do with it.

[jekyll-install]: https://jekyllrb.com/docs/installation/windows/
[docsify-install]: https://docsify.js.org/#/quickstart?id=quick-start

I'm not exactly sure how a wiki is going to fit into this whole thingy, but I like the idea of developing inside a container. It gives the whole thing a very *Homogenize* feel since each teammate's work PC will be more implicit than, say, installing all the prerequisite dev tools on their work PC like we typically do. Easy enuf, except, <mark>I need to bake in development environments as part of source repos.</mark> And that means [Dev Containers][dev-containers-quick-start]!

[dev-containers-quick-start]: https://docs.docker.com/language/nodejs/develop/

The first thing we're going to need is a dead simple [node container][docker-images-nodejs-official] to run [docsify-template][docsify-template-repo] locally, so a little `docker run` magic is all that's necessary to [sync the repo contents][docker-cli-map-source-dir] and spawn the container. Then I can [run `npx serve` to spin up an HTTP server][npm-serve] to preview on localhost and docsify will parse the markdown to generate the site on the fly for me. Kids' stuff!

[docker-images-nodejs-official]: https://hub.docker.com/_/node/tags?page=1&name=bullseye
[docsify-template-repo]: https://github.com/docsifyjs/docsify-template
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

It's slightly obnoxious that docker runs in the foreground and would require another Terminal to stop. It would be impossible to run in [detached `-d` mode][docker-cli-detached-mode] without giving away docker [clean up `--rm` option][docker-cli-clean-up] completely. And you do want docker to automatically clean up when the container exit. It's time to switch to compose.

[docker-cli-detached-mode]: https://docs.docker.com/engine/reference/run/#detached--d
[docker-cli-clean-up]: https://docs.docker.com/engine/reference/run/#clean-up---rm

I prefer `docker compose` over `docker`. They keep everything in a YAML file, so you don't need to memorize what ports to expose, the working directory to map, args to override, and the like. All I need to do is break out the same `docker` command that I use to spawn the container in detached `-d` mode to [create a `compose-dev.yml` file][docker-compose-create-yml] and we're all set.

[docker-compose-create-yml]: https://docs.docker.com/get-started/08_using_compose/#create-the-compose-file

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

Then I can use `docker compose` commands to spawn the container and docker will do the rest for me.

```bash
#!/bin/zsh
# .dotfiles/.functions

# Spin up the container in detached mode
dc up -f compose-dev.yaml
# Stop and remove the container
dc d -f compose-dev.yaml
```

> **Note**
>
> `dc up` and `dc d` are shell scripts written for my sanity so I can more quickly get back to coding. See [my .dotfiles repo on GitHub][gh-dotfiles-repo] to learn more.

[gh-dotfiles-repo]: https://github.com/kosalanuwan/dotfiles/#readme

The other thing we're going to need is an editor. [I use Typora for markdown][hello-world] but we're going to need JavaScript and Html/Css support to change the default theme. And that means VS Code, but there are more problems. Working without extensions is intense. Not only is there no intellisense, but no language recognition at all. They require node runtime to function. I'm going to go ahead and say that they don't have access to node runtime inside the container, so they have no way of knowing what syntax highlighting and intellisense to show. And you do want to have syntax highlighting and intellisense to work. Time to install the [Remote - Containers][vscode-remote-containers-extension] extnesion and `Attach to Running Container`. Then I can install the required extensions for node and JavaScript. It's slightly obnoxious that VS Code doesn't install recommended extensions from the `.vscode/extensions.json` automatically, so I'm going to have to install `recommendations` manually.

[hello-world]: https://keepontruckin.hashnode.dev/hello-world
[vscode-remote-containers-extension]: https://aka.ms/vscode-remote/download/containers

Unfortunately, there's no simpler way of converting word docs and slide decks into markdown so I'm going to have to copy all the contents from the individual directories that are in. And there's no way I'm going to go to 100 pages and slides to create markdown content one at a time, so it's definitely necessary to push this initial content to a git repo and let the teammates contribute.



/KP



> **PS:** For the complete work, see [my devcontainers-try-docsify repo on GitHub][gh-repo].

[gh-repo]: https://github.com/alertbox/devcontainers-try-docsify/tree/devenvs-try-docsify#readme
