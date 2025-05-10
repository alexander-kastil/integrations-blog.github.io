+++
date = '2025-05-08T13:22:53+02:00'
title = 'Setup a Hugo based Blog on GitHub Pages'
draft = false
+++

I'm starting a new blog to share my technical knowledge, hosted on GitHub Pages. I chose Hugo as the static site generator along with the elegant [Clarity theme](https://github.com/chipzoller/hugo-clarity). Let me walk you through the setup process:

## Base Setup

Download and install [Hugo CLI](https://gohugo.io/installation/)

Create the Hugo blog directory using Hugo CLI:

```bash
hugo new site integrations-blog
cd integrations-blog
git init
git submodule add https://github.com/chipzoller/hugo-clarity themes/clarity
echo theme = 'clarity' >> hugo.toml
hugo server
```

Add the first post:

```bash
hugo new posts/setup-hugo-blog.md
```

Edit the post in content/posts/setup-hugo-blog.md:

```markdown
... add some content ...
```

Create a base menu in hugo.toml:

```toml
[menu]
  [[menu.main]]
    name = "Blog"
    url = "/posts/"
    weight = 1

  [[menu.main]]
    name = "Classes"
    url = "/classes/"
    weight = 2

  [[menu.main]]
    name = "Services"
    url = "https://www.integrations.at"
    weight = 3
```

Add padding to the code blocks in `_syntax.scss`:

```SCSS
code
...
    padding-left: 1rem
```

Create a repo `GITHUB_USERNAME/'URL.github.io` ie `alexander-kastil/integrations.github.io` and update the base URL in `hugo.toml`:

```toml
baseURL = 'https://alexander-kastil.github.io/'
```

Rebuild the Hugo site:

```bash
hugo server -D
```

Read through the deployment options, Change to folder `public` and commit the changes:

```bash
cd public
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/alexander-kastil/integrations.github.io.git
git push -u origin master
```

## Deployment

Next we will set up the GitHub Pages deployment. Go to the repository settings and enable GitHub Pages. Set the Build and Deployment source to GitHub Actions and search for the Hugo action. Select the latest version and add it to your repository. This will automatically build and deploy your Hugo site to GitHub Pages whenever you push changes to the main branch.

![hugo-action](images/hugo-action.jpg)

## Custom Domain

I am using Microsoft DNS to manage my custom domain. I created a CNAME record pointing to `alexander-kastil.github.io` and four a records pointing to the following IP addresses:

- 185.199.108.153
- 185.199.109.153
- 185.199.110.153
- 185.199.111.153

![dns](images/dns.jpg)

Update the following line in the `hugo.toml` file:

```toml
baseURL = 'https://blog.integrations.at/'
```

Last but not least, add the custom domain in the GitHub Pages settings and enforce HTTPS. This will ensure that your custom domain is properly configured and linked to your GitHub Pages site.

Now check the deployment status in the GitHub Actions tab. If everything is set up correctly, you should see a successful deployment message. You can also check the GitHub Pages settings to ensure that your custom domain is properly configured and linked to your GitHub Pages site.

![dns](images/check-deploy.jpg)
