---
title: How to Create Personal Blog on github Using Hexo
date: 2021-08-11 03:34:13
tags: 
- Hexo
- Blog
---

## Dependencies
+ Node.js
+ npm
+ Git


## Install Hexo
if you don't have `node.js` and npm `npm` on your workstation, please visit [official site](https://nodejs.org/en/) to install the node.js. (npm would be automatically installed if node.js installation is done)
To check whether node.js and npm are successfully installed:
```bash
# in cmd/terminal
node -v # v14.17.3 or others
npm -v # 6.14.13 or others
``` 
If you are using win10 as your operate system and get errors with these codes, you should re-open the powershell/cmd as administor after the installation then try the codes above.

Then:
```bash
npm install -g hexo-cli
```
To check whether hexo successfully installed:
```bash
hexo -v
```
You'll see the version information e.g.:
```bash
 hexo-cli:4.3.0, 
 node:14.17.3, 
 ... 
```

## Create Blogs
First of all, you have to navigate to the the folder, where you plan to create your blogs' folder.
```bash
cd E:\
mkdir my_blogs
cd my_blogs
```
Then run:
```bash
# if you are using powershell as administor on windows,
hexo init
# if you are on mac/linux, 
sudo hexo init
```
Create your first post:
```bash
hexo new "My First Post"
```
A new file with .md format will be created in `source/_posts/`. You could directly edit it in file. Also, you could preview your blog on your local machine with:
```bash
#establish a localhost for blog
hexo server

# if you have some updates to your blogs, use
hexo clean
hexo generate
hexo server
```
The response will show you a addresse like `http://localhost:4000`, you could visit it on local.


## Deploy Blogs
After you know how to write and make changes to your blogs on local, we'll introduce how to create a github page and deploy your blog to it. Install Deloyer with:
```bash
npm install -save hexo-deployer-git
``` 
Go to your github and create a new repositery called `username.github.io`. `username` should be your github username. This will be used as the address of github page later.

On your local, you need to edit the `_config.yml`. You could find this file in the blog folder. `_config.yml` is the key to set up your blogs, such as set up the name, discription, theme and so on. 
```bash
# in _config.yml, find the part #Deployment change the info like below
deploy:
  type: 'git'
  repo: your-repo-adress
  branch: master
```
After this, in your command line type:
```bash
hexo deploy
```
After a while, you would be able to visit `username.github.io` to see your own blog.

## Change the Theme of Your Blog
There are varous free themes on [Hexo's official site](https://hexo.io/themes/). You could pick one to visit its github repo and follow the instruction to install it.

E.g. I'd like to change the theme of my blog, firstly, 
```bash
# in the path of your blog folder, clone the theme to local
git clone `url_of_the_theme_repo` themes/my_theme
```
Then modify the `_config.yml`. (most theme repos have demo, you could directly compare the `_config.yml` to your own and make some changes)

After this:
```bash
hexo clean
hexo generate
# take a preview on local, 
hexo server
# if everything is fine,
hexo deploy
```



you've successfully created your own blog! Congrats!