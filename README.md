# My website

Built using the [remote theme template](https://github.com/mmistakes/mm-github-pages-starter) for the [Minimal Mistakes Jekyll theme](https://github.com/mmistakes/minimal-mistakes).

I've made the following custom adjustments:

- I'm using a simple landing page with a "Welcome" message instead of the theme's default home page layout (i.e. one with a "recent posts" feed). See [here](https://github.com/mmistakes/minimal-mistakes/issues/2349) or [here](https://github.com/mmistakes/minimal-mistakes/issues/2191#issuecomment-504080616). This also meant that I moved the paginator for recent posts to `blog/index.html`. See [here](https://github.com/mmistakes/minimal-mistakes/issues/2191#issuecomment-504080616). 
- Added [academic icons](https://jpswalsh.github.io/academicons/) to the site author section (LHS) of the page. Basically, this involved downloading the icon CSS set and copying across `academicons.css` and `academicons.min.css` to the `css/` directory. The icons can then be referenced and used in the `_config.yml` file (e.g. "ai ai-google-scholar" for the Google Scholar icon).
- Changed the default syntax highlighting (i.e. the embedded code chunks) to *Solarized light*. See [here](https://mmistakes.github.io/minimal-mistakes/docs/stylesheets/#syntax-highlighting) and [here](https://github.com/mmistakes/minimal-mistakes/issues/2278). 
- Speaking of code chunks, I've also added R **blogdown** functionality for writing and converting .Rmd posts with integrated code. See [here](https://bookdown.org/yihui/blogdown/jekyll.html).

This is relevant only to my site &mdash; I'm writing it down as a reminder for myself &mdash; but I also ran the following changes after updating from my old version of Minimal Mistakes to fix some post aesthetics. The latter were themselves relics leftover after converting my original [blogger website](https://stickmanscorral.blogspot.com/).

```sh
cd _posts
sed -i '/^layout:/d;/^date:/d;/^modified_time:/d;/^blogger_id:/d' *.md *.html
sed -i 's/\/images\/post-images/\/assets\/images\/post-images/g' *.md *.html
sed -i 's/\\(-\\)/\&mdash;/g' *.html
sed -i 's/<u><a href/<a href/g' *.html
sed -i 's/<\/u><\/a>/<\/a>/g' *.html
```
