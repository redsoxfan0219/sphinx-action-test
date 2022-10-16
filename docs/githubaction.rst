Overview of GitHub Actions (Written in reStructureText)
=======================================================

Introduction to State Site Generators
-------------------------------------

The purpose of static site generators is two-fold:

- To allow uses to write in a simple, lightweight markup language (typically Markdown or reStructuredText)
- To render that simple markup text into attractive, stylized HTML files

The beauty of static site generators is that, after some initial setup, that process is reduced to a single command line statement, like the one I use to build the HTML for this site:

:: 
    sphinx-build -b html docs/ docs/_build

As you can see, this statement only requires one to specify the source for the markup files, the destination of the outputs, and the format of those outputs.

Shortcoming of Static Site Generators
-------------------------------------

While convenient in many ways, static site generators are not perfect on their own.

For example, imagine you are building a documentation template that will be used by a large team. You want everyone to write in the Markdown or reStructuredText files you've built, and you want everyone to render those files into HTML when finished, and finally you want them to host those HTML files as a GitHub Pages site that is attached to their code repository.

You're not worried about the team writing in Markdown. But you don't want to worry about everyone rendering the files into HTML, a process many of them may be unfamiliar with. What if they don't remember to add the rendered files into the correct output destination? What if the forget to add that required ``-b`` flag? 

Most of all, you want to make *your* life easier. If someone screws up something in the above, that person's coming to you with an issue. An issue that you don't have time or focus to worry about.

GitHub Actions, a CI/CD Tool
----------------------------

Generally, it's a good idea to reduce the number of times users have to do the same action manually. Automation is almost always preferable. With the HTML build process, we have an excellent candidate for automation. But how do we do it?

The answer is `GitHub Actions <https://github.com/features/actions>`_.

Introduced a few years back, GitHub Actions is a mechanism that can be used to introduce automate workflows. It's grown to become a key function for many teams' `continuous integration, continuous deployment (CI/CD) <https://www.redhat.com/en/topics/devops/what-is-ci-cd>_. 

Under the hood, GitHub Actions works by receiving directions you write in a ``.yml`` file, which you post within your code repository under a ``./.github/workflows`` subdirectory. This file provides instructions for instantiating and directing a virtual machine.

Using a GitHub Action in Docs-as-Code Deployment
------------------------------------------------ 

For our purposes, GitHub Actions is valuable because it can be used to automate the HTML build process and commit that content to your GitHub repository.

The best part? It's super simple.

This page was built using a GitHub Action I put together in 20 minutes. You can find that ``.yml`` file `here <https://github.com/redsoxfan0219/sphinx-action-test/blob/main/.github/workflows/docs_pages.yaml>_. I've also reproduced the contents of that file below.

Explanation of the GitHub Action .YML 

Below are the contents of the file I wrote for my GitHub Action. I provide a walkthrough of the contents further down. 

::

name: docs_pages_render

on:
  push:
    branches:
      - main

jobs:

  build_docs_job:
    runs-on: ubuntu-latest
    env: 
      GITHUB_PAT: $ {{ secrets.GITHUB_TOKEN }}

    steps: 
      - name: Check out the main branch to the VM
        uses: actions/checkout@v2.3.4

      - name: Set up Python in the VM
        uses: actions/setup-python@v2.2.1
        with:
          python-version: 3.10
      
      - name: Install dependencies
        run: |
          python -m pip install sphinx
          python -m pip install myst-parser
          python -m pip install sphinx_togglebutton
          python -m pip install sphinx-copybutton
      - name: Render HTML
        run:
          sphinx-build -b html docs/ docs/_build

      - name: Set up temporary repository and commit to HTML files
        run: 
          cd _build
          git init
          touch .nojekyll
          git add .
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git commit -m 'Deploy rendered HTML'

      - name: Push rendered HTML to destination branch
        uses: ad-m/github-push-action@v0.5.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: gh-pages
          force: true
          directory: ./docs/_build

Here are the steps performed as a result of the ``.yml`` file above:

1. When the repository's event monitor detects a ``git push`` to the ``main branch``, GitHub stands up a Linux (with Ubuntu distribution) virtual machine (VM).

2. The VM checks out the ``main`` branch of the repository.

3. The VM sets up Python v3.10.

4. The VM downloads (via ``pip``) all necessary dependencies to build the HTML files.

  - These dependencies are all outlined in the repository's ``docs/conf.py`` file.

5. The VM renders the HTML from the reStructuredText and Markdown files.

6. The VM changes directory into the ``_build`` output directory.

7. The VM initiates a Git repository within the ``_build`` output directory.

8. The VM stages and commits the newly rendered HTML files, using a default git commit message.

9. Finally, the VM pushes the committed files back to my repository and deploys the HTML to the ``gh-pages`` branch.

And voilà! We have our rendered and deployed HTML (which you are now viewing).



