# blog-hugo
Hugo based blog for tigrou.nl

My help:
http://gohugo.io/tutorials/github-pages-blog/
http://gohugo.io/overview/quickstart/

# build files for deployment + deploy to github
hugo --baseUrl=http://www.tigrou.nl --theme=hyde-x
./deploy.sh

# run server locally
hugo server --theme=hyde-x --buildDrafts
# or in watch mode
hugo server --watch -t hyde-x

