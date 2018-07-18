---
title: new drasss
tags:
---

# 草稿

## 私密文件，舍不得删的时候放进来


草稿相当于很多博客都有的“私密文章”功能。

会在source/_drafts目录下生成一个new-draft.md文件。但是这个文件不被显示在页面上，链接也访问不到。也就是说如果你想把某一篇文章移除显示，又不舍得删除，可以把它移动到_drafts目录之中。

    $ hexo new draft "new draft"

如果你希望强行预览草稿，更改配置文件：

    render_drafts: true

或者，如下方式启动server：

    $ hexo server --drafts

下面这条命令可以把草稿变成文章，或者页面：

    $ hexo publish [layout] <filename>
