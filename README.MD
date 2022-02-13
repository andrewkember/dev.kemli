# Hugo static site for dev.kem.li

## Resources

* <https://gohugo.io/hosting-and-deployment/hosting-on-github/>
* <https://github.com/peaceiris/actions-hugo>
* <https://gohugo.io/content-management/urls/#yaml-front-matter>
* <https://bwaycer.github.io/hugo_tutorial.hugo/extras/aliases/>
* <https://domchristie.github.io/turndown/>
* <https://medium.com/swlh/are-you-using-svg-favicons-yet-a-guide-for-modern-browsers-836a6aace3df>
* <https://www.regisphilibert.com/blog/2018/02/hugo-the-scope-the-context-and-the-dot/>
* Tips for modules <https://www.hugofordevelopers.com/articles/master-hugo-modules-managing-themes-as-modules/>
* https://jakearchibald.github.io/svgomg/
* Light/dark themed SVG favicon <https://web.dev/building-an-adaptive-favicon/>
* Search: <https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-features/#search-page>
* Custom 404: <https://moonbooth.com/hugo/custom-404/>

So the branch stuff...

Aim:

* Full code on `main` branch
* Generated Hugo content in `public` directory published to `gh-pages` orphan branch
* Feature branches from main for development
* Drafts are maintained with `draft = true`. No need to use a branching strategy with drafts.

[Turndown](https://domchristie.github.io/turndown/) seems to be a quick way to convert simple HTML to Markdown, which is nice.

Lots to do for the move to PaperMod:

- [ ] Custom 404
- [ ] Represent text/banner
- [ ] Make HTTPS a GitHub Pages thing, not a Cloudflare thing??
- [ ] Migrate to blog.kember.net.