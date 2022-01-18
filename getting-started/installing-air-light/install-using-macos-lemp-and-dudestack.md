---
description: >-
  How to use Air-light in a native macOS LEMP environment with a modern
  WordPress development stack based on roots/bedrock.
---

# Install using macOS LEMP & dudestack

### Using Air-light with our WordPress stack and development environment

This is an option we at here [Dude agency](https://github.com/digitoimistodude) use.

{% hint style="info" %}
**Please note:** This way requires many packages and is a bit of work to get up and running. We recommend you read all the documentation thoroughly. You can always use [your own setup and try option 2](https://github.com/digitoimistodude/air-light/wiki/Using-with-dudestack-and-LEMP#option-2-use-in-your-own-wordpress-instance-and-with-your-own-development-environment-like-docker-local-by-flywheel-or-mamp).
{% endhint %}

### Install

1. Set up [macos-lemp-stack](https://github.com/digitoimistodude/macos-lemp-setup#installation-steps) by following the instructions on GitHub.
2. Ensure that [http://localhost](http://localhost) works and \~/Projects is linked to /var/www and both exist
3. Install [dudestack](https://github.com/digitoimistodude/dudestack#installation) by following instructions provided in the repository
4. Install [debuggers for gulp and for your editor](https://github.com/digitoimistodude/air-light#debuggers)
5. Create a new project by using dudestack's `createproject` command (explained [here](https://github.com/digitoimistodude/dudestack#starting-a-new-project-with-createproject-bash-script))
6. From Terminal go into **bin** folder: `cd /path/to/where/you/cloned/air-light/bin` and run `sh newtheme.sh`. This script takes care of the rest as it updates textdomain with your project name, checks updates for air and npm packages, runs npm install, fetches devpackages, sets up gulp, cleans up the leftover files and activates the theme via WP-CLI.
7. Go back to project directory `cd ~/Projects/yourproject`, run `gulp` and start coding!

\
