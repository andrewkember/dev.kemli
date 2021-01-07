# Hugo static site for dev.kem.li

## Resources:

* https://gohugo.io/hosting-and-deployment/hosting-on-github/
* https://github.com/peaceiris/actions-hugo


So the branch stuff...

Aim:

* Full code on `main` branch
* Generated Hugo content in `public` directory published to `gh-pages` orphan branch
* `dev` branch from main for development
* `draft` branch for drafts, PR into main to publish <- this might be a bad idea - means front matter will always be `draft = false`. Depends on how I end up pubishing. 