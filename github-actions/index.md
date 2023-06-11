# Github Actions for Hugo Page



This short article provides how to deploy a [Hugo](https://gohugo.io/) site where the github actions come in handy. (It is not a tutorial to build a site with Hugo)

Before the technical part, I would like to demystify the motivation for building this site which, as depicted, should act like a chronicle of thoughts for reflecting, documenting and organizing sparks of inspiration over time. Additionally, it serves as a portfolio that exhibits a diverse range of work. 

Now it's time for github actions. 
## Theme
The selected theme runs as a submodule in the source repo so that Hugo is able to build the site from the container by solely copying the markdown files. 
<code>git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt</code>

## Deployment
{{< admonition >}}
There are two repos in my implementation: one is to store markdown and config files and the other is for publishing the static site. 
But you can definitely achieve this by one repo as described in [Hugo Docs](https://gohugo.io/hosting-and-deployment/hosting-on-github/) 
{{< /admonition >}}

Thanks to [actions-hugo](https://github.com/peaceiris/actions-hugo), we have a simple solution for deploying a Hugo site with github actions. 
1. Generate deploy keys by
<code>ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f deploy-key -N ""</code>\
With the key pair, we store private key and public key correspondingly in source repo (Actions secrets and variables) and page repo (Deploy keys). 
2. Put workflow file in .github/workflows/ in source repo. 
```yaml
name: Remote Deployment

on:
  push:
    branches:
    - master

jobs:
  build-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@master
      - name: Checkout submodules
        shell: bash
        run: |
          auth_header="$(git config --local --get http.https://github.com/.extraheader)"
          git submodule sync --recursive
          git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --force --recursive --depth=1        
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 0.68.3
          extended: true
  
      - name: Build
        run: hugo --minify
  
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.DEPLOY_KEY }}
          external_repository: Ykaros/ykaros.github.io
          publish_branch: master  
          publish_dir: ./public
```




