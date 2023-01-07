---
title: "How to Create a Blog Using Hugo"
author: "Guillaume Legoy"
date: "2023-01-07"
---

{{< toc >}}


## Objective of this Article

Getting started with a blog can be daunting. Like with any technology dozen of options are available and it's easy to get discouraged by not knowing where to start. Therefore as much as this article is about writing down my learnings for later use and avoid forgetting them, I hope maybe it can be one day useful to someone who wants to get started with their own blog and provide some guidance. If I can do it, so do you.


## What is Hugo?

[Hugo](https://gohugo.io/) is a static website generator framework. Static websites are sites whose content is fixed and doesn't change. Consequently these sites are rapid to build, deploy, and load, but are also more secure and easy to regenerate if needed. Thus they are the perfect solution for blogs that don't need dynamic capabilities. 

As a framework Hugo does all the heavy lifting and generates the necessary HTML and CSS for you so you can focus on your content. If you wish to explore the static website generator landscape before making your choice, [Jekyll](https://jekyllrb.com/), [Pelican](https://getpelican.com/), [Eleventy](https://www.11ty.dev/), or [Gatsby](https://www.gatsbyjs.com/) to only name a few, are also great options. [Substack](https://substack.com/inbox) is also a popular one where you only have to write and don't have to setup anything.

Finally, Hugo content such as this blog post is written using [Markdown](https://www.markdownguide.org/), a lightweight markup language that is ideal to create formatted text using a plain-text editor or IDE such as [VS Code](https://code.visualstudio.com/) which is the one I use. It makes writing easy and was paramount in my choice to use Hugo.


## Hugo Themes

One of the advantages of Hugo is the vibrant community that provides us with many themes. A theme is how your blog looks and feels. There are hundreds of them, such as [Academic](https://academic-demo.netlify.app/), [Papermode](https://github.com/adityatelange/hugo-PaperMod), and many others, ranging from minimalistic blogs to complex transactional websites. The theme I use for this blog is [Hermit](https://github.com/Track3/hermit). Choose a theme based on your requirements. For my blog I needed:

- a minimalistic theme
- a simple way to display blog posts

Therefore it doesn't matter to me if the theme offers few customization options or hasn't been updated in a while. On the other hands if you require many options such as SEO, photo display, or other advanced capabilities, pick a theme accordingly.

Lastly a warning: while Hugo themes are great, customizing them using CSS and HTML can result in a massive headache. Documentation is sparse at best and you'll be lucky to find a resolution to your issue on forums. You'll need to be willing to hack your way through CSS and HTML files and use the [page inspector functionality](https://firefox-source-docs.mozilla.org/devtools-user/page_inspector/) of your brower to modify your theme to your liking. To me this is kind of fun, but be warned. New Hugo versions can also break themes if they weren't updated in a while. That being said, I will log all my findings in the last section of this blog on how to change the look and feel of the Hermit theme so that you can get started easily if you decide to choose the same theme.


## Prerequisites

Before we start, make sure the following tools are installed on your computer:

- [install Hugo](https://gohugo.io/installation/)
- [install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [read the Hugo "Getting Started" documentation](https://gohugo.io/getting-started/quick-start/)
- [read the Hermit theme documentation](https://github.com/Track3/hermit)

## Generate the Blog Skeleton 

Using the command line, we will now create our website skeleton. To do so execute the following commands:

```bash
cd ~/Documents # Pick the location (here ~/Document) you want your blog folder to be located in.
hugo new site my-cool-blog # Pick the name of your folder here (my-cool-blog for this tutorial).
cd my-cool-blog
git init
```

At this point it's time to add the theme folder. 

To add a theme (for this example I'll use Hermit), run the follwing:

```bash
git submodule add https://github.com/Track3/hermit.git themes/hermit
echo "theme = 'hermit'" >> config.toml
```

The above command will create a folder in the `themes` folder and then add the theme to the `config.toml` file to tell Hugo we want the Hermit theme to be used. Your blog is now ready to be built locally, run 

```bash
hugo server
```

and access your blog in a brower at `http://localhost:1313/`. Any change will automatically be picked up.


## Deployment with Netlify

Now that our blog is ready, it's time to make it available to everyone on the internet. To deploy our website we will use [Netlify](https://www.netlify.com/), a website deployment solution. Netlify makes this step extremely simple, and Hugo provides with a [great step by step tutorial](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/) on how to do it. Be aware you will be charged around 15$ a year for this service. If you want something free, [Github Pages](https://pages.github.com/) are a great alternative.

Note that at this point your site will be accessible through the `your-site-name.netlify.app` domain. If you wish to own your own domain (like `boringdatascience.com`), you will need to purchase a domain name. I use [Namecheap](https://www.namecheap.com/) but any solution out there will do. 


## Customization

Now comes the fun part. Each theme as far as I know behaves differently with Hugo. Which is why you will find many support post on Hugo forum asking why X or Y theme isn't working and Hugo maintainers unable or unwilling to answer as it's not part of Hugo core. What follows is proper to the Hermit theme, so if you aren't using it you can probably stop reading and refer to your own theme documentation, if it exists (fun!). Nevertheless some insights might still apply to your theme, so if you are stuck, check what's following.

### Styling

To change the look and feel of your blog, you'll need to edit css files. To do so you first need to copy the content of `themes/hermit/assets/scss` into `assets/scss` (always starting from the root of your site folder). The reason is we don't want to directly change the theme css files, but instead change their copies. These copies we just created will override the original. It's also easy to revert any unwanted change if needed that way.

Once that is done, we can edit our blog style. Keep in mind I'm not good at css so I mainly hack my way through trial and error. Feel free to comment and tell me what I'm doing is awful and I should be banned from approaching a website ever again. With that being said, I'll now list a non-exhaustive list of what can be changed, always starting with the path of the file to look at:

#### Text and colors

In `themes/hermit/assets/scss/style.scss`, search for the `html` tag and modify as desired, such as:

```scss
html {
  background: $light-grey;
  line-height: 1.5;
  font-weight: 400;
  font-size: 120%;
  letter-spacing: -.01em;
  scroll-behavior: smooth;
  p {
    margin-bottom: 1rem;
    margin-top: 1.5rem};
  h1 {
    color: #265483;
    margin-top: 2em;
    margin-bottom: 2em;
    font-weight: 700;
    font-size: 140%;
  }
  
  h2 {
    color: #265483;
    margin-top: 2.5em;
    margin-bottom: -0.5em;
    font-weight: 600;
    font-size: 120%;
  }

  h3, h4, h5, h6 {
    color:  #333;
    margin-top: 2.5em;
    margin-bottom: -0.5em;
    font-weight: 600;
    font-size: 110%;
  }
}
```

This controls a lot of things, especially space between paragraphs and under titles, which help your text breathe and don't look too cramped.


You may also have noticed in the above snippet that we have a `$light-grey` parameter. In `themes/hermit/assets/scss/_predefined.scss`, you can find and change colors. Mine look like this (yes I know colors don't match their name but I'm lazy):

```scss
// Colors
//
$theme: #d55c1a;
$text: #333;
$light-grey: #ffffff;
$dark-grey: #e07c18;
$highlight-grey: #7d828a;
$midnightblue: #433e56;
```


Finally, still in `themes/hermit/assets/scss/_predefined.scss` you can change links color as well as links highlight colors when hovering over them (e.g. [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/) in orange and green):

```scss
@mixin aTag {
  a {
    color: $theme;

    &:hover {
      color: #20c997;
    }
  }
}
```

#### Code blocks

This one gave me a massive headache. If you want to change the background color of code blocks such as the one below, first open `themes/hermit/assets/scss/style.scss` and find the `pre` class under `code`, then you need to add the `!important` tag to override Hugo's default. I still haven't found out how to change code comments, so drop me a line if you figure it out.

```scss
code ,
pre tt {
  font-family: $code-fonts;
}

pre {
  padding: .7em 1.1em;
  overflow: auto;
  font-size: 0.9em;
  line-height: 1.5;
  letter-spacing: normal;
  white-space: pre;
  background: #433e56 !important;
  border-radius: 4px;
  margin-bottom: 3rem;
  // -webkit-overflow-scrolling: touch;

  code {
    padding: 0;
    margin: 0;
    color: #fff !important;

 }
}
```

#### Page width

By default I find Hugo's text column width to be too narrow, which doesn't provide a nice reading experience. To change this default, find the `.thin` class in `themes/hermit/assets/scss/style.scss` and modify the `max-width` parameter:

```scss
.thin {
  max-width: 1000px;
  margin: auto;
}
```

### Hyperlinks

If you want links such as [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/) to open in a new window, do the following:

1) Create the following file (and parent folders if needed): `layouts/_default/_markup/render-link.html`
2) Copy and paste the following snippet in the newly created file:

```html
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{ .Text | safeHTML }}</a>
```

### Table of Content

Tables of contents are imo very nice to have as they provide more clarity to blog posts and allow for easy navigation on subsequent visits (because I know you keep re-reading my posts, right?....right?). To add one, follow these steps:

1) Create the following file: `layouts/shortcodes/toc.html`
2) Copy paste the following HTML code in the newly created file:

```html
<div>
    <h2> Table of Content </h2>

</br>

    {{ .Page.TableOfContents }}
</div>
```

3) In `themes/hermit/assets/scss/style.scss` find the `#TableOfContents` class and change it to your taste:

```scss
#TableOfContents {
  font-size: 1em;
  //@include dimmed;

  ul {
    padding-left: 0em;
    margin: 0;
  }

  &>ul {
    list-style-type: none;

    ul ul {
      font-size: 1em;
    }

    li li {
      padding-left: 1.5em;
      list-style-type: none;
    }

  }
}
```

### Social Icons

To add social icons to the main welcome page of your blog, simply add the following in you `config.toml` file found in your website root or in `config/_default`:

```markdown
 [[params.social]]
    name = "twitter"
    url = "your-twitter-profile-url"

  [[params.social]]
    name = "linkedin"
    url = "your-linkedin-profile-url"

  [[params.social]]
    name = "github"
    url = "your-github-profile-url"
```

Happy coding!