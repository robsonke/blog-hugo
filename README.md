## Tigrou.nl blog
Blog for tigrou.nl, build with http://gohugo.io/

### Publish
# build files for deployment + deploy to github
hugo --baseUrl=http://www.tigrou.nl --theme=hyde-x

### New post

1. Write the post in markdown and save it to `content/posts/the-url-I-want-it-to-have.md`
2. Or generate it with Hugo: hugo new post/2015-01-01-something-to-blog.md

Make sure that all assets of the post (images, videos etc) are saved to `static/the-url-of-the-blog-post/some-image.png`

#### previewing:
hugo server --watch -t hyde-x
#### or non listening
hugo server --theme=hyde-x --buildDrafts

It will be accessible at `localhost:1313`
