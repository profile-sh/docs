---
layout: page
title: Jupyterlite Demos
description: 
giscus_comments: true
author: {{site.author}}
date: 10-12-2024
---

 [Jupyterlite](https://jupyterlite.readthedocs.io/en/stable/) is currently under development, some functionality may not work as expected. We will probably need iterative updates on this project as it develops. For the moment let us just summarize the main points.

There are several kernels already available in Jupyterlite, especially for python there is jupyterlite-xeus and jupyterlite-pyodide. As of today: 
- The xeus option does not allow installation of python packages after the site is deployed, while pyodide does.
- The xeus option requires two separate environment config files, pyodide needs only one.
- The RTC is not working if we use latest Jupyter.

## Project structure:

In the root folder of our project repo, we need the following files and folders:

- environment.yml, defines python environment
- jupyter_lite_config.json, build-time config
- jupyter-lite.json, app config
- contents (name should match the build-time config), a folder for the static content (e.g. notebooks) we are going to serve.

Optionally, we can have separate jupyter-lite.json files, for each app we build (e.g. repl, tree), under the folders named after the apps (e.g. repl/jupyter-lite.json). The option to select the apps we want to build can be set as a list under the key "apps" in the jupyter_lite_config.json file.

As of today, for Jupyterlite-xeus:

- in addition to environment.yml, we also need build_environment.yml, otherwise the *contents* folder will not show up on the deployed site.
- the build command, *jupyter lite build* implicitly picks the environment.yml (if we change the file name, we have to make it explicit (--XeusAddon.environment_file) in the build command)
  
Demo repos: 

- [using pyodide](https://github.com/jupyter-ed/jupyterlite-pyodide)
- [using xeus](https://github.com/jupyter-ed/jupyterlite-xeus/tree/main)

Acknowledgements and References:

Jupyterlite is a great addition to the Jupyter-family, the following resources were very helpful in writing this document:

1. [Jupyterlite docs](https://jupyterlite.readthedocs.io/en/stable/)
2. [Jupyterlite repo](https://github.com/jupyterlite/jupyterlite)
3. [Jupyterlite-pyodide demo](https://jupyterlite.github.io/demo/lab/index.html)
4. [Jupyterlite-pyodide demo repo](https://github.com/jupyterlite/demo)
5. [Jupyterlite-xeus docs](https://jupyterlite-xeus.readthedocs.io/en/stable/)
6. [Jupyterlite-xeus demo](https://jupyterlite.github.io/xeus-python-demo/lab/index.html)
7. [Jupyterlite-xeus demo repo](https://github.com/jupyterlite/xeus-python-demo)

