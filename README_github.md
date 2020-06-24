# GitHub Actions for automagically deploying your website

This final part will be about getting GitHub actions to automagically deploy your website. See the [previous part](README_sphinx.md) to set up the `Doxygen`/`Sphinx`/`Breathe` pipeline first.

The resulting website will be **public to the web**, even if the project is private. This is a limitation of GitHub. If you would like you can use GitLab instead, which lets you host a password protected private website for documenting your private repo, which is pretty cool!

## Git

If you haven't already, initialize a `git` repo for your project:
```
git init .
```
A good `.gitignore` would be:
```
.vscode/
.DS_Store
build/
_build/
_static/
_templates/
```
With these I was able for our toy project to just
```
git add .
git commit -m "Initial commit"
```

Obviously we need a GitHub repo, so go ahead and make one. I called mine `cpp_doxygen_sphinx` (surprise). See the note above if you are making it a private repo.

Use the instructions to push your initial commit to GitHub.

## GitHub Actions

`GitHub Actions` are fancy because there is a great marketplace of predefined actions for us to use, so we don't have to play around with it too much.
`GitHub Actions` are **not** fancy because there is no good way to test your actions "offline" (except [here](https://github.com/nektos/act), but it's kind of a pain).

We are going to use [this great Deploy to GitHub Pages action](https://github.com/marketplace/actions/deploy-to-github-pages). Go ahead and navigate there, click use latest. You can see the snippet that we will use.

To set this up, go to our main directory and make two new directories
```
mkdir .github
mkdir .github/workflows
```
All the workflows will live in here.

Make the workflow for generating our documentation - it should be a new file `.github/workflows/docs.yml`.

Open `docs.yml` and edit it such that it reads
```
name: Docs

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
  
jobs:
  build:

    runs-on: macos-latest

    steps:
    - name: Requirements
      run: brew install doxygen
        && brew install sphinx-doc
        && pip install sphinx-rtd-theme
        && pip install breathe
    - name: Build docs
      run: cd docs_sphinx
        && make html
    - name: Deploy
      uses: JamesIves/github-pages-deploy-action@releases/v3
      with:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        BRANCH: gh-pages # The branch the action should deploy to.
        FOLDER: docs_sphinx/_build/html # The folder the action should deploy.
```
Breaking it down:
* `Requirements`: Some stuff comes pre-installed on the image, some doesn't. Luckily `brew` and `pip` do, but `doxygen` and `sphinx` and such - well you are on your own! The complete list of pre-installed software is [here](https://github.com/actions/virtual-environments/blob/master/images/macos/macos-10.15-Readme.md).
* `Build docs`: This builds the docs just like before.
* `Deploy`: This deploys the `docs_sphinx/_build/html` folder.

Add it to the `git` repo and push:
```
git add .github
git commit -m "Docs"
git push
```