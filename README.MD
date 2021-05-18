# Hugo static site for dev.kem.li

## Resources:

* https://gohugo.io/hosting-and-deployment/hosting-on-github/
* https://github.com/peaceiris/actions-hugo
* https://gohugo.io/content-management/urls/#yaml-front-matter
* https://domchristie.github.io/turndown/


So the branch stuff...

Aim:

* Full code on `main` branch
* Generated Hugo content in `public` directory published to `gh-pages` orphan branch
* `dev` branch from main for development
* `draft` branch for drafts, PR into main to publish <- this might be a bad idea - means front matter will always be `draft = false`. Depends on how I end up pubishing. 

I had to override the `_default/baseof.html` template to fix the Google Analytics call (for v4, which doesn't differentiate the async from the sync script). So I should check the diff on that frequently :-(.

Using the theme as a module sounds great, and may be the future of Hugo, but either I don't understand the precedence rules correctly, or it's not very useful. Going for submodules. Rig for dive. Open all main vents.  

The URLs links above shows how to do url aliases. I'd need that if I want to redirect old urls (Pelican) to new (Hugo)

Turndown seems to be a quick way to convert simple HTML to Markdown, which is nice. 

Ansible:
* Ansible lint looks useful