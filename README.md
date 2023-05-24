# Marco's Tech Blog

## Where to find me
https://blog.marcozh.com/

## How to build and run locally
On first clone, pull the submodules used for this blog
```bash
git submodule update --init --recursive
```

Build the hugo site
```bash
hugo
```

Serve the site locally
```bash
hugo serve
```

## How to add new content
Use the hugo command to create a new post
```bash
hugo new content/post/<post-name>.md
```

## How to deploy
The blog is hosted on github pages, and the deployment is done via github actions.
To deploy, simply push to the master branch and the action will take care of the rest.


## Technology used
- [Hugo](https://gohugo.io/)
- Github Actions
- Github Pages
- [Discus](https://disqus.com/)
