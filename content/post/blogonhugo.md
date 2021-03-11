---
title: "Migrated my blog out of Blogger"
date: 2021-02-10T18:07:46+11:00
draft: false
tags: ["blog", "hugo", "github", "github actions", "JAMStack"]
---
## Problem  
While blogger is an amazing platform for hosting your blog, with perks of easy linking custom domains and google analytics integration, the main issue is with styling/rendering of your website and most of all blogger "owns" the content and if its decides it can just take it out or make parts paid. I had bitter experience with another hosting platform, where my account was in-accessible and had to spend hours writing to customer support with little support from them . So decided to make the big move.

## Requirement
- Wanted full control of content and styling 
- Do not want to spend hours creating/editing HTML files while adding content
- Quick and easy content addition if not as easy as adding a post in blogger

## Options
1. Wordpress hosting  
   Hosting on wordpress was easy on platforms like wordpress.com with options for themes, easy adding of posts. But here too the content was at the mercy of the hosting platform. You don't have full control of the styling unless you have the knowhow of editing wordpress themes and styling. Also the free tier does not give custom domain option and you cannot use custom theme unless you pick a business hosting option.  
2. Github Pages  
   Github is where I spend most of my hobby time adding/forking repo and committing the code that I play around with, so it felt like home to maintain a blog too on this platform. Also github pages support custom domains and also comes with an SSL option. The HTMLs have to be uploaded or generated using a static website generator. JAMStack is something that I have been long waiting to try, so Option 2 was my choice from all its angles without a second thought.

## Technology Choices
**Jekyll** was the first option as it was suggested by github pages, but it put me off right at the beginning when it asked to setup a ruby environment on my windows machine. If it was a .Net based environment, i would have been happy as i spent most of my time building on Microsoft stack. I just wanted a simple CLI to do the job for me, unfortunately I couldn't find a .Net based static site generator.   

[**statiq**](https://statiq.dev/) is an interesting option but its slightly different from what i was looking for and i would definitely  give it a try some time in the future. All the others are either dead or not maintained. I have forked an old archived .net based generator who knows i might resurrect it.

The next best option was [**hugo**](https://gohugo.io/), it was easy setup, tons of features and building custom theme was super easy. So **hugo** it is.

## Implementation  

Straight away created a github repository added new hugo website files, chose a theme and it was all up and running in no time.

Content is written as markdown file for easy and consistent formatting. 
Hugo uses the markdown files and the html templates to generate the html files. Chose a Hugo theme called [Github Style](https://github.com/MeiK2333/github-style) which resembles the github dashboard page, some customization is still underway.
CI/CD used behind is github actions, which is triggered for each commit. Actions downloads hugo cli and generates the website and pushes the generated html to a separate branch gh-pages which is configured as the branch for github pages.
Custom sub-domain name www.beneathabstraction.com is used along with https to have the URL setup. traffic to domain beneathabstraction.com is redirected to the subdomain route using an ALIAS entry at the domain provider.

Migrating the content from blogger was not as straight forward, but with a bit of work it can be achieved. I did not have many blog entries as most of my old blogs where lost with the other hosting provider blocking me off. But migrated all that I had in blogger.
- Downloaded the blogger backup, which downloads an atom format XML. 
- Opened the XML in browser and copied just the posts XML section into another XML file  
- Added an [XSLT](https://gist.github.com/focalstrategy/1989435) to the post XML, and open it in browser 
- Copy pasted the content and updated the styling and the images paths to a local static folder path.
  
## Missing features  
Hosting a static website comes with it own drawback like comments and share feature is missing, contact me page has to be built. But that were API in JAMStack comes into play. With some Javascript and API calls the missing features can be implemented. Thats something to be looked at for the future.
