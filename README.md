## Terms & Conditions

_config.yml          ← configures the site (set your username and repo)
Gemfile              ← for local development
_layouts/default.html ← layout with EN | ES | PT language switcher
assets/css/style.css  ← clean legal-style design

index.md                          ← home page with app list
apps/myapp/index.md               ← table of links per language
apps/myapp/terms/{en,es,pt}.md    ← T&C in 3 languages
apps/myapp/privacy/{en,es,pt}.md  ← Privacy Policy in 3 languages
apps/myapp/{terms,privacy}/index.md ← automatic redirects

To deploy to GitHub Pages:
1. Create a repo on GitHub, push this folder to main
2. Settings → Pages → Source: main / / (root)
3. Edit _config.yml: set your [YOUR-USERNAME] and if it's a project site (not user site) add baseurl: "/repo-name"

Before publishing to the App Store, find and replace all [PLACEHOLDER] values:
- [APP NAME], [DEVELOPER 1], [DEVELOPER 2], [CONTACT EMAIL], [DATE], [CITY]

To add another app: copy apps/myapp/ → apps/new-app/, update the front matter and add the entry in index.md.

Liability is capped at the purchase price of the app — this is the standard for paid indie apps and passes App Store review.   