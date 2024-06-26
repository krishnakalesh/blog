+++
title = 'Hosting Hugo generated site in Github 🌊'
date = 2024-03-08T16:49:25+05:30
draft = false
+++
# Hosting Hugo generated site in Github 🌊

In this post, we'll explore how to create a website using Hugo, a static site generator, and host it on GitHub Pages. We'll also integrate the GitHub-style theme for a sleek and modern look.
<!--more-->
The website taken as an example is a blog site here. Also, we wil add Gitalk integration for capturing comments from blog vistors.

## What is Hugo?

[Hugo](https://gohugo.io/) is a fast and flexible static site generator written in Go. It allows you to create static websites quickly and easily, with support for themes, content management, and more.

## What is Github Pages?

[Github Pages](https://pages.github.com/)  is designed to host your personal, organization, or project pages from a GitHub repository.
Simplicty is to the level - it can be hosted directly from your GitHub repository. Just edit, push, and your changes are live.

## Gitalk
 
[Gitalk] (https://github.com/gitalk/gitalk) is a modern comment component based on GitHub Issue and Preact.

## Let's create the repositories first in github

For the whole process we need to create two repositories. 
    
1. First one is where we curate our website content (say - 'blog', i will call this as 'blog' repo through out below explainations).
2. Second one is the repository which will act as the website (say - 'gamezone', i will call this as 'gamezone' repo through out below explainations).

So, idea is that the repo 'blog' will act as the place where we curate our website contents and once we are ready we can build
the code to create the contents. The contents genetaed will be copied to the repo 'gamezone' and contents will be available for 
viewers. 
 
##Install Hugo
 
Steps for installing Hugo available @https://gohugo.io/installation/
Once Hugo installed you can finalize theme for your site from the list - https://themes.gohugo.io/
 
## Setup contents
 
    1. Do git clone of your blog repo, say - https://github.com/xyz/blog.git

    2. Once cloning comeplete do 
    > cd blog
    > hugo new site gamezone (this command create new Hugo site)
    > cd gamezone

    3. Next step is to add a theme to site. Say, we selected a theme (github-style/) 
    > cd themes
    Then, follow the steps specified in theme setup - https://themes.gohugo.io/themes/github-style/   
    
    4. We need to configure hugo.toml with details like baseURL, languageCode, title, theme
    A sample content of toml file can be found at - https://themes.gohugo.io/themes/github-style/
    The base URL will be in the format (https://{YOUR GITHUB ID}.github.io/{YOUR WEBSITE REPO NAME}/)

    5. Once site created, theme copied; we can verify the site by bringing up the site in local 
    using command
    > hugo server   

   This command should be execute from base folder 'gamezone' and it will bring up site in URL: http://localhost:1313/

## Setup New Post
 
    > hugo new post/POST_NAME.md
    
## Edit the post with your custom content
 
   As we mentioned early, blog repo is the place used for storing the code and gamezone repo is the where we deploy from.
   For deployment easiness, we can add gamezone repo as submodule of blog repo. Before adding gamezone repo 
   as submodule create atleast one branch for 'gamezone' repo and do atleast one commit to that branch (say branch name is 'main').

    > cd /blog/gamezone
    > git submodule add -b main https://github.com/xyz/gamezone.git public

Now, when we do hugo build, the static content will go inside 'public' folder of /blog/gamezone and automatically the contents of 'public' folder will be get bound to the repo https://github.com/xyz/gamezone.git

## Build 

    > cd /blog/gamezone
    > hugo -t {NAME_OF_THEME} (Example: hugo -t github-style)
    > cd public
    > git remote -v (just ensure remote is https://github.com/xyz/gamezone.git)
    > git status
    > git add .
    > git commit -m 'init repo'   
    > git push origin main

![Alt text](/pinchofcode/images/hugo1.png)

## Deployment
 
   After git push, you can go and verify the https://github.com/xyz/gamezone.git and we 
   can notice that all sthugo aic content generated by hugo build available in git repo
   This will automatically gets deployed. 
   
   Verify the github pages settings from  Settings--> Pages option of the git repo.
   ThE URL to access  the site is also available here.

![Alt text](/pinchofcode/images/hugo2.png)

## Gitalk  
 
  For recording site vistors comments we can use https://github.com/gitalk/gitalk
  Security for comments can be set up using https://github.com/settings/applications/new
  Many theme templates already will have this facility, otherwise add below code to capture 
  comments for your posts.
  
```
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
 
	<div id="gitalk-container"></div>
 
	<script type="text/javascript">
		var gitalk = new Gitalk({
			clientID: 'GitHub Application Client ID',
			clientSecret: 'GitHub Application Client Secret',
			repo: 'your GitHub repo name',
			owner: 'GitHub repo owner',
			admin: ['GitHub repo owner and collaborators, only these guys can initialize github issues'],
			id: decodeURI(location.pathname), // Ensure uniqueness and length less than 50
			distractionFreeMode: false  // Facebook-like distraction free mode
		})
 
		gitalk.render('gitalk-container')
    </script>
```

## Common Issues

1. Make sure the 'baseURL' in hugo.toml file is coreect. This can be different in local vs production.
2. gitalk configuration in hugo.toml, specify id = "location.pathname"
3. For gitalk configuration, inside Github -> Settings ->  Developer settings -> [YOUR OAUTH APP] 
   Authorization callback URL should be set as your website base URL.

