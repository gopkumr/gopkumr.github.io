# Repository for my personal blog
Github Pages Deployment: ![Github Pages Deployment](https://github.com/gopkumr/gopkumr.github.io/actions/workflows/GithubPagesBuild.yml/badge.svg?branch=main)


This repository contains all the content and files required to generate a static website for the my personal blog.   
**The blog uses the below technology stack**
- **Content** is written as markdown file for easy and consistent formatting. 
- **Website Generator** used is *Hugo* which uses the markdown files and the html templates to generate the html files. A *Hugo* theme called [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) is used to generate the website.
- **CI/CD** used behind is github actions, which is triggered for each commit. Actions downloads hugo cli and generates the website and pushes the generated html to a separate branch *gh-pages* which is configured as the branch for github pages.
- **Custom** sub-domain name *www.beneathabstraction.com* is used along with *https* to have the URL setup. traffic to domain *beneathabstraction.com* is redirected to the subdomain route using an ALIAS entry at the domain provider.
