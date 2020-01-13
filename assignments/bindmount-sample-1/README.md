# Bind Mounts Using Jekyll site
Jekyll "static site generator" starts a local web server that uses Markdown for blog posts. The project is dependent on the image listed below, but copied to `rrmhearts/jekyll-serve` for safekeeping.

```
docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
```