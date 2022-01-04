# This is my homepage hosted on GitHub.

## how to maintain this repo:

1.  clone repo:

```bash
git clone git@github.com:VivianKeith/VivianKeith.github.io.git
```

2. update the `next` theme with option `--recurse-submodules`  when clone repo, or use cmd `git submodule`  after cloning repo:

   - cloning:

   ```bash
   git clone git@github.com:VivianKeith/VivianKeith.github.io.git --recurse-submodules
   ```

   - cloned:

   ```bash
   git submodule init
   git submodule update
   ```

2. install necessary npm modules:

```bash
npm install hexo //on windows
npm install hexo-cli //on linux or MacOS
npm install
```

3. add post article:

```bash
hexo new "title of a new article"
cd source/_posts/
vim "title of a new article".md
```

4. deploy the updated blog:

```
hexo clean
hexo g
hexo d
```

5. update repo:

```bash
git add .
git commit -m "some description"
git push origin hexo
```



## how to custom the theme:

modify `_config.yml` in root path and `_config.yml` in `.themes/next`，according to the hexo official manual and `hexo-next-theme` official guidance. 👇👇👇

- hexo official doc: https://hexo.io/zh-cn/docs/index.html

- next -theme official doc: https://theme-next.js.org/docs
- old-version-next-theme doc: http://theme-next.iissnan.com/

The `next-them` itself is a git repo, as a [submodule](https://www.yiibai.com/git/git_submodule.html) when located in the homepage main repo. Therefore, after the  `_config.yml` in `.themes/next` is modified, use `git add` and `git commit` to update the [submodule repo](https://zhuanlan.zhihu.com/p/87053283) in `.themes/next`,  then commit changes in root path via `git add` and `git commit`.

## more info:

https://github.com/hexojs/hexo

https://github.com/iissnan/hexo-theme-next

https://github.com/theme-next/hexo-theme-next
