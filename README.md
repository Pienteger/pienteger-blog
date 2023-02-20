# Pienteger Blog

## Configuration

Within gatsby-config.js, you can specify information about your site (metadata) like the site title and description to properly generate meta tags.

```js
// gatsby-config.js

module.exports = {
  siteMetadata: {
    title: `Gatsby Starter Glass`,
    author: {
      name: `Yinka Adedire`,
      summary: `self-taught front-end dev. jamstack enthusaist.`,
    },
    description: `A Minimal & Beautiful Gatsby Personal Blog Starter With Nice Glassmorphism Ui.`,
    siteUrl: `https://gatsbyglass.netlify.app`,
    social: {
      twitter: `yinkakun`,
    },
  },

  // ...
};
```

## Manually Editing contents

### Blog Posts

Blog contents can be updated in markdown format at `content/blog`. Delete placeholder posts and start blogging.

```md
---
title: Do you know anything about sensory deprivation tanks?
date: 2021-02-02
tags:
  - stranger things
  - tv series
  - "2022"
social_image: /media/rocket.jpg
description: This is a custom description for SEO and Open Graph purposes. If
  it's not provided, it defaults to auto-generated excerpts of the page content.
---

This top portion is the beginning of the post and will show up as the excerpt on the homepage.
```

### Pages

Homepage intro, Contact, and About page content can be updated in Markdown format at `content/pages`.

```md
---
title: About Mee
template: about-template
profile_image: /media/profile-image.jpg
---
```

# Editing Contents with Netlify CMS

This project is preconfigured to work with Netlify CMS.
When Netlify CMS makes commits to your repo, Netlify will auto-trigger a rebuild / deploy when new commits are made.
You’ll need to set up Netlify’s Identity service to authorize users to log in to the CMS.

- Go to <https://app.netlify.com> > select your website from the list.
- Go to Identity and click Enable Identity.
- Click on Invite Users and invite yourself. You will receive an email and you need to accept the invitation to set the password.
- Now headover to Settings > Identity > Services and Enable Git Gateway.
- You can also manage who can register and log in to your CMS. Go to Settings > Identity > Registration  Registration Preferences. I would prefer to keep it to Invite Only if I am the only one using it.
- Now, go to to site-name.netlify.app/admin/, and login with your credentials.

Once you are in your Netlify CMS, you can navigate to Posts and Pages. Here you will find a list of existing pages and posts.
