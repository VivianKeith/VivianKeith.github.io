# This is my homepage hosted on GitHub.

## how to maintain this repo:

1.  clone repo:

```bash
git clone git@github.com:VivianKeith/VivianKeith.github.io.git
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

- next -theme official doc: https://theme-next.js.org/docs/getting-started/
- old-version-next-theme doc: http://theme-next.iissnan.com/

❗ **NOTE:**

The `next-them` itself is a git repo. However, it is so troublesome to maintain homepage main project and theme sub project at the same time. Therefore, the `.git` in `.themes/next` is deleted so that the `next-them`  becomes a normal directory which can be commit together.

If there is a need to maintain two projects separately, we can refer to:

- https://sharifxu.top/2020/05/14/Hexo-Mutiagent-Manager/
- http://w4lle.com/2016/06/06/Hexo-themes/index.html
- http://andersjing.com/2018/06/30/hexo-theme/
- https://github.com/iissnan/hexo-theme-next/issues/932



## more info:

GitHub site of hexo and next theme：

https://github.com/hexojs/hexo

https://github.com/iissnan/hexo-theme-next

https://github.com/theme-next/hexo-theme-next

maintain hexo homepage on multi clients：

https://zhuanlan.zhihu.com/p/87053283





