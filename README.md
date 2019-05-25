# blog

source for <https://blog.comicat.me>

theme use [comicat-hu/hexo-theme-cafe](https://github.com/comicat-hu/hexo-theme-cafe) (forked from [giscafer/hexo-theme-cafe](https://github.com/giscafer/hexo-theme-cafe))

## Getting Starting

### clone roject

* `git clone https://github.com/comicat-hu/blog -b source --single-branch`
* `git submodule update --init --recursive`

### install dependencies

* `npm install hexo -g`
* `npm install`

### basic hexo command

* `hexo g` (generate files in public folder)
* `hexo s` (run local server on port 4000)
* `hexo d` (deploy project follow _config.yml deploy setting)
* `hexo clean` (clean all files in public folder, recommand clean after modify any framework and theme file)

### write a post

* `hexo new "Post Title"` (create new post)
* set categories and tags
* use `<!--more-->` to set post preview
* use `{% asset_img Picture_Name Picture_ALT %}` to set a picture in post
