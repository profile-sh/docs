---
layout: page
title: GitHub Pages
description: A basic project on how to create and deploy GitHub pages
giscus_comments: true
author: {{site.author}}
date: 18-11-2024
---

Documenting work is very important. If we do not document our work, with the passage of time we forget the steps and process that helped us achieve our goal. Also, documentation helps showcase work, collaborate with others, and makes it easier for others to build upon our dreams after we leave. It also helps improve writing. [GitHub pages](https://docs.github.com/en/pages) is an easy to use service for this purpose. 

GitHub pages is a static website deployed on GitHub, free of charge, thanks to GitHub. The pages are used for documentation (for everything related to open source projects on GitHub) and GitHub user profiles.  In this project, we are going to learn how to create and deploy multi-repo GitHub pages under a GitHub organization, with comments functionality enabled using [giscus](https://giscus.app/). We will also learn how to setup [GitHub actions and workflows](https://docs.github.com/en/actions) for building and deploying the pages. 

## 1. Setup GitHub Pages

In this section we will learn how to setup GitHub pages for repos under an organization.

### 1.1 Setup a GitHub organization

The purpose of creating an organization is to isolate all the repos we will need in this project. You will be the only member of the organization, there is no need to add other members for now.
- Exercise: Login into your GitHub account and create a free public organization with a name of your choice, for example, gh-pages-project. For help, visit [GiHub docs](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch).

### 1.2 Setup a GitHub pages repo 

We now have the organization named gh-pages-project (or the organization name that you picked while creating the organization). we need a repo named [your organization name].github.io under this organization. This repo is *required* for deploying GitHub pages.
- Exercise: Go to your organization page (https://github.com/[your_organization_name]), click on repositories, and create a new public repo with *your organization name* as the owner and the repo name [your organization name].github.io. Use *add a readme* option, .gitignore option (select jekyll from dropdown), and for license use MIT license.

If all of the steps were successful, visit the site at https://[your organization name].github.io and you see your static website hosted on GitHub!

### 1.3 Add a theme

Our website layout is very basic so far. To improve it, we need to use a theme. Github Jekyll based [supported themes](https://pages.github.com/themes/) make this step very simple. To use the theme, we need to create a new file at the root of our repo.

- Exercise: Create a new file named _config.yml in the root folder of the repo [your organization name].github.io with the following content:

{% raw %}     
```yaml
title: GitHub Pages
description: Bookmark for updates!
remote_theme: pages-themes/cayman
plugins:
- jekyll-remote-theme
```
{% endraw %}     

Where *cayman* is the name of the theme we want to use. You may use any of the [supported themes](https://pages.github.com/themes/).

### 1.4 Create your first page

So far we have been using our README.md file for our homepage. Let us create our first page with some content in it.

- Exercise: From root folder of your repo ([your organization name].github.io), create a new file with the path pages/home.md, paste the following content in it, and commit changes:

{% raw %}       
```yaml
---
layout: default
title: GitHub Pages
description: Create and deploy GitHub pages
permalink: /
---
This project is about GitHub pages.
```
{% endraw %}     

Wait for a minute or so as GitHub rebuilds your site, visit the site (https://[your organization name].github.io) and refresh your browser, you will see the updated content.

### 1.5 Enable GitHub pages for another repo

Sometimes we have multiple projects in several repos and we want to keep the documentation where it belongs. It would be great if we have a way to show all our documentation on a single website. GitHub makes it very easy, below we go through the steps needed to enable the pages from a different repo. The process may be used for any new repos you create under the organization.

- Exercise: Go to the homepage (https://github.com/[your_organization_name]) of the organization for this project and click on *repositories* link and create a new public repository with the name project-repo (or any name you like) under the organization for this project. Use options for creating readme.md, .gitignore (jekyll), and MIT license.

- Exercise: Enable the GitHub pages for the new repo you have created. Visit the repo homepage and:
  - click *settings* 
  - click *pages* on the left sidebar
  - under Build and Deployment, find the label "branch" select *main* and change */(root)* to */docs*, then click *save*.

The last step above is necessary for enabling pages for a repo, it was not needed for the repo [your organization name].github.io as the pages are enabled by default for a repo with the name *.github.io.

- Exercise: Create a page in the new repo:
  - To add a theme, create a file with the path docs/_config.yml in the root folder of the repo (named project-repo) and paste the following content in it:

{% raw %}     
```yaml
title: project-repo docs
description: Docs for the project-repo
remote_theme: pages-themes/cayman
plugins:
- jekyll-remote-theme
```
{% endraw %}     

  - Create a file with the path *docs/home.md* and paste the following content in it after editing the web links according to the name you picked for your organization:

{% raw %}     
```yaml
---
layout: default
title: Docs for the project-repo
description: Docs for the project-repo
permalink: /
---
This is the docs homepage for the repo [project-repo](https://github.com/gh-pages-project/project-repo).
```
{% endraw %} 

Visit the link https://[your organization name].github.io/project-repo and if everything went well you will see the homepage for the  project repo docs. Well done! 
Before we take a coffee break, let us create a link to the above page: 
- Exercise: Create a link to the project-repo docs on the homepage of the repo [your organization name].github.io:
  - Go to the repo [your organization name].github.io and add the following line at the bottom of the file pages/home.md (replace gh-pages-project with the name of your organization:
    
```
This project is about GitHub pages, visit the [project document]({% link pages/gh_pages_project.md %}) to learn how the project 
was implemented.

Visit the example repo [example](https://github.com/gh-pages-project/project-repo) with a docs folder as the source of pages [pages for example repo docs](https://gh-pages-project.github.io/project-repo/).
```

Wait for a minute or so and visit the  page https://[your organization name].github.io/ and click on the link you have just created. 
Great, let us take a coffee break.

## 2. Implement Comments 

Taking comments from others about your work is a step towards collaboration and it also helps improve your work. Let us implement comments functionality for our site.

In the subsections below, you will see some code using html, Liquid, and Jekyll [jekyll docs](https://jekyllrb.com/docs/). In this project we are not going to explain the code, that is beyond our current scope, we just want to implement comments here.

### 2.1 Enable discussions for the pages repo 

Enabling discussions is a requirement to allow comments on our pages. For a demo, We will enable discussions only for the pages repo [your organization name].github.io, this will allow us to enable comments in the pages we create in the repo. 

- Exercise: Enable discussions for the pages repo
  - visit you pages repo [your_organization_name]).github.io and click *settings*
  - under *Features*, select *Discussions* option to enable discussions.

Now discussions are enabled, next step is to install [giscus app](https://giscus.app).

### 2.2 Install giscus app for the organization

We are going to use giscus app (it is free and open source) as a backend for the comments functionality. We will install the app at the organization level so that we can use it for multiple repos if the need arises.

- Exercise: Install [giscus app](https://giscus.app):
  - visit your organization page (https://github.com/[your_organization_name])
  - click *settings*
  - near the bottom of side menu, click *GitHub Apps*
  - click *GitHub Marketplace*
  - in the search field, type *giscus* and click on the search result *giscus*
  - at the bottom of page above the green button *Install it for free* select your organization
  - click *Install it for free*
  - click  *Allow my billing information to be linked with this organization*
  - click *save*
  - click *Complete order and begin installation*
  - under *Repository access*, select *Only select repositories* and pick the pages repo [your_organization_name]).github.io from dropdown choices.
  - click *install*

We are done with installing the giscus app. In the next exercise, we are going to generate html code that we need to include in our pages to allow comments.

### 2.3 Include giscus generated html code

The html code we need is unique for every repo. We can generate it for any or every repo under our organization. For our demo, we need it only for our pages repo [your organization name].github.io.
- Exercise: generate html code using giscus app
  - visit [giscus app](https://giscus.app)
  - scroll down to "Configuration*, select the *language* for comments
  - under *repository*, type [your organization name]/[your organization name].github.io
  - under *Page ↔️ Discussions Mapping*, select *Discussion title contains page URL*
  - under *Discussion Category*, select *Announcements*
  - under *Features*, select "Enable reactions for the main post"
  - under *Theme*, select a theme
  - under *enable giscus* copy the generated html code, in my case it is:

{% raw %}     
```html
<script src="https://giscus.app/client.js"
    data-repo="gh-pages-project/gh-pages-project.github.io"
    data-repo-id="R_kgDONOp0jQ"
    data-category="Announcements"
    data-category-id="DIC_kwDONOp0jc4CkRnZ"
    data-mapping="url"
    data-strict="0"
    data-reactions-enabled="1"
    data-emit-metadata="0"
    data-input-position="top"
    data-theme="preferred_color_scheme"
    data-lang="en"
    crossorigin="anonymous"
    async>
</script>
```
{% endraw %} 

  - visit the pages repo root folder , create a new file with the path _includes/giscus.liquid and paste the code you have copied in the previous step
  - wrap your code in a div as follows and commit changes:

{% raw %}     
```html
<div style="max-width: {{ site.max_width }}; margin: 0 auto;">
code generated by giscus app here
</div>
```
{% endraw %} 

### 2.4 Create a page layout in the pages repo

- Exercise: Visit the root folder of the pages repo:
  - create a new file with the path _layouts/page.liquid
  - paste the following code and commit changes:

{% raw %} 
```html
---
layout: default
---
<h1>{{ page.title }}</h1>
<p>{{page.date}}</p>
<p>{{page.author}}</p>
{{content}}   
{% if page.giscus_comments %}
{% include giscus.liquid %}
{% endif %}
```
{% endraw %} 

### 2.5 Create a test page for comments

- Exercise: Visit the root folder of pages repo:
  - create a new file with the path pages/test_comments.md
  - paste the following content in it and make changes for author name and date fields and commit changes:
 
{% raw %}     
```yaml
---
layout: page
title: A page with comments enabled
description: test giscus comments
giscus_comments: true
author: author_name
date: 17-11-2024
---

This is a test page with comments enabled.
```
{% endraw %} 

Let us create link for the test_comments page for easy navigation:

Exercise: visit the root folder of the pages repo, edit the file pages/home.md by adding the following content at the bottom of the page:

{% raw %} 
```liquid
A page for testing giscus comments functionality is [here]({% link pages/test_comments.md %}).
```
{% endraw %} 

Note: The two methods used above for linking a page use different file paths for the same page, *test_comments.md vs pages/test_comments.md". In this introductory project we will not discuss this subject (links and linking in Jekyll)). A tutorial on Jekyll and Liquid syntax is beyond the scope of this project.
Great, we are done with enabling the comments. 

### 2.6 Enable discussions for the organization

Enabling discussions for a repo is a requirement to allow comments on our pages. We can also optionally enable discussions  at the organization level, that will be a place to discuss the organization level topics.

- Exercise: Enable discussions for the organization
  - under your organization, create a public repo named *.discussions* to host discussions 
  - visit your organization page (https://github.com/[your_organization_name]) and click *settings*
  - near middle of the left side menu, click *Discussions*
  - select *Enable discussions for this organization*, select *[your organization name].discussions* for the repo to host discussions, and click *save*

Great, we now have a place to discuss everything that comes under our organization. Next let us implement a GitHub workflow for our pages project. Break time.

## 3. Setup a GitHub workflow 

GitHub [actions and workflows](https://docs.github.com/en/actions) help us automate CI/CD. In fact, in this project, we are already using a GitHub workflow behind the scenes. Whenever, we make changes to a file in the pages and project repos ([your organization name].github.io and project-repo), GitHub runs a work flow to build and deploy the pages. If we click on *Actions* tab for one of the repos, we will see a list of runs of the workflow *pages build and deployment* under *All workflows*. This workflow runs whenever we commit to the main branch of our repo (the runs are listed under actions tab for each repo with pages enabled). 

The default workflow mentioned above is nice but it is not in our hands. Let us create our own workflow to build and deploy GitHub pages (see [docs](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages) for more info):

### 3.1 Enable custom workflows for a repo
The following exercise will disable the default workflow and enable our custom workflow to build and deploy pages.
- Exercise: Enable custom workflows for the repo pages
  - click settings of the repo [your organization name].github.io
  - click *Pages* on the left side menu
  - under *Build and deployment*, select *Github Actions*  under *source*

### 3.2 Write a custom workflow

- Exercise: Write a workflow to build and deploy GitHub pages
  - visit the root folder of the pages repo [your organization name].github.io
  - create a new file with the path .github/workflows/build_deploy_gh_pages.yml
  - paste the following content in it and commit changes:

{% raw %}    
```yaml
name: build_deploy_gh_pages
on: 
  workflow_dispatch

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Jekyll
        uses: actions/jekyll-build-pages@v1
        with:
          source: ./
          destination: ./_site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./_site
  # Deployment job
  deploy: 
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{steps.deployment.outputs.page_url}}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
{% endraw %}
    
### 3.3 Run the custom workflow

The above workflow will not run automatically. Every time we want to rebuild the pages after making changes, we will need to run the workflow manually as follows:
- click *Actions* tab of the repo
- on left side menu click build_deploy_gh_pages
- click the dropdown *Run workflow* on the right side
- wait for the run to finish
- refresh your pages site and if everything went well, the site will reload successfully

So far we have implemented a custom workflow only for the repo [your organization name].github.io. We will need to repeat the process for our other repo named project-repo:
Exercise: Follow all steps in section 3 above for the repo named project-repo. The only change we need to make is the value of source field under actions/jekyll-build-pages@v1 in the workflow code: change "./" to "./docs".

There is one improvement we should make. We have almost the same code for the workflows in both of the repos ([your organization name].github.io and project-repo). It would be nice if we can create a reusable workflow that both repositories can call. Let us do that now.

### 3.4 Reusable workflow

We need to create a new repo that will host the reusable workflow that both of our repos can call, then create a reusable workflow in it and change the code of the repo workflows we have created above so that we can call the reusable workflow from both repos.

- Exercise:
  - create a public repo named *workflows* under our organization (follow the usual steps to create a repo under our organization).
  - in the root folder of the *workflows* repo, create a new file with the path ./github/workflows/build_deploy_gh_pages.yml, paste the following code in it and  commit changes:

{% raw %} 
```yaml
name: build_deploy_gh_pages reusable workflow
on:
  workflow_call:
    inputs:
      build_source:
        description: source directory for build
        required: false 
        type: string
        default: ''
      build_destination:
        description: destination directory for build
        required: false 
        type: string
        default: _site
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: checkout file
        uses: actions/checkout@v4          
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: jekyll-build-pages@v1
        uses: actions/jekyll-build-pages@v1
        with:
          source: ${{inputs.build_source}}
          destination: ${{inputs.build_destination}}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ${{inputs.build_destination}}
  deploy_gh_pages:
    needs: build
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{steps.deployment.outputs.page_url}}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
{% endraw %} 
        
- Exercise:
  - switch to the repo [your organization name].github.io
  - open the ./github/workflows/build_deploy_gh_pages.yml file to edit and replace the content with the following code and commit changes:

{% raw %}            
```yaml
name: build_deploy_gh_pages
on:
  workflow_dispatch 
jobs:   
  job1: 
    permissions:
      contents: read
      pages: write
      id-token: write
    name: call the reusable workflow
    uses: gh-pages-project/workflows/.github/workflows/build_deploy_gh_pages.yml@main
    with:
      build_source: './'
```
{% endraw %} 

Now let us test our new setup. Click on *Actions* tab and run the updated workflow. Wait for the workflow to finish. If it finishes successfully, go to your pages site and refresh your browser. If all went well you will see your site.

We need to repeat the above steps for our repo named project-repo with the following change in the workflow code: Change the value of build_source from './' to './docs'. Also test the workflow.

## 4. Including Code, Images, Diagrams, and Math  

This topic is a bit advanced for this project, a separate project is needed for this topic. Just a quick rundown:

### 4.1 Images

Include a local image ([image source](https://unsplash.com/photos/sun-light-passing-through-green-leafed-tree-EwKXn5CapA4)):

```
![image]({% link data/img3783small.jpeg %})
```

result:

![image]({% link data/img3783small.jpeg %})

### 4.2 Code blocks

Code blocks are enclosed inside triple backticks with the coding language name appended as below:
  
```
        ```html
        <p> html code example </p>
        ```
```

the above code renders as:

```html
<p> html code example </p>
```

### 4.3 Diagrams and math

We use [Mermaid](https://mermaid.js.org/) for diagrams, and  [MathJax](https://www.mathjax.org/) for math equations. 
See test pages ([test_mermaid]({% link pages/test_mermaid.md %}) and [test_mathjax]({% link pages/test_mathjax.md %})). See the source code in the *_includes* and *pages* folders of the [repo](https://github.com/gh-pages-project/gh-pages-project.github.io).

Great, the project is done. Happy writing ❤️.

Acknowledgments: I got a lot of help from [github docs](https://docs.github.com) and [jekyll docs](https://jekyllrb.com/) for writing this document. 
