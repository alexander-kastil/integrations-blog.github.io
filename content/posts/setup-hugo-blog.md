+++
date = '2025-05-08T13:22:53+02:00'
draft = true
title = 'Setup a based Hugo Blog on GitHub Pages'
+++

I'm starting a new blog to share my technical knowledge, hosted on GitHub Pages. I chose Hugo as the static site generator along with the elegant [Clarity theme](https://github.com/chipzoller/hugo-clarity). Let me walk you through the setup process:

Download and install [Hugo CLI](https://gohugo.io/installation/)

Create the Hugo blog directory using Hugo CLI:

```bash
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/chipzoller/hugo-clarity themes/clarity
echo theme = 'clarity' >> hugo.toml
hugo server
```

Add the first post:

```bash
hugo new posts/my-first-post.md
```

Edit the post in content/posts/my-first-post.md:

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
