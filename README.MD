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

So the branch stuff...

Aim:

* Full code on `main` branch
* Generated Hugo content in `public` directory published to `gh-pages` orphan branch
* Feature branches from main for development
* Drafts are maintained with `draft = false`. No need to use a branching strategy with drafts. 

I had to override the `_default/baseof.html` template to fix the Google Analytics call (for v4, which doesn't differentiate the async from the sync script). So I should check the diff on that frequently :-(.

Using the theme as a module sounds great, and may be the future of Hugo, but either I don't understand the precedence rules correctly, or it's not very useful. Had to use submodules for Ananke theme. Trying Go modules again for Coder. Rig for dive. Open all main vents.  

The URLs links above shows how to do url aliases. I'd need that if I want to redirect old urls (Pelican) to new (Hugo).

[Turndown](https://domchristie.github.io/turndown/) seems to be a quick way to convert simple HTML to Markdown, which is nice.

Lots to do for the move to Coder:

- [ ] Logo top left
- [ ] About + Contact page
- [ ] Navigation and archive
- [ ] Initial navigation from splashpage
- [ ] Fix low contrast code block typeface in dark mode
- [ ] Get the template overrides workng again (twitch, twitch, is this what kills go modules again?)
