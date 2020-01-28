# The Hacker-Blog theme

*Hacker-Blog is a minimalistic, responsive jekyll theme built for hackers. It is based on the [hacker theme](https://github.com/pages-themes/hacker) for project pages.*

[Demo](https://ashishchaudhary.in/hacker-blog)

### Included

1. Pagination
2. SEO tags
3. Archive Page
4. RSS
5. Sitemap 

## Local Build

If you want to see the changes before pushing the blog to Github, do a local build.

1. [`gem install jekyll`](https://jekyllrb.com/docs/installation/#install-with-rubygems)
2. `gem install jekyll-seo-tag`
3. (`cd` to the blog directory, then:) `jekyll serve --watch --port 8000`
4. Go to `http://0.0.0.0:8000/` in your web browser.

*Note: In case you have set a `baseurl` different than `/` in `_config.yml`, go to `http://0.0.0.0:8000/BASEURL/` instead.*

### Local build using docker

```bash
docker run --rm -p 8000:8000 \
  --volume="LOCATION_OF_YOUR_JEKYLL_BLOG:/srv/jekyll" \
  -it tocttou/jekyll:3.5 \
  jekyll serve --watch --port 8000
```

Replace `LOCATION_OF_YOUR_JEKYLL_BLOG` with the full path of your blog repository. Visit `http://localhost:8000/` to access the blog.

