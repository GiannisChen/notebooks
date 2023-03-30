## 在GitHub上搭建个人博客

- https://docs.github.com/zh/pages/quickstart



### 简明步骤笔记



#### 在Windows上安装jekyll

1. 在Windows上安装`jekyll`，前往https://jekyllrb.com/docs/installation/windows/找到安装包，下载`Ruby`和`Devkit`作为Windows上的依托https://rubyinstaller.org/downloads/。
2. 根据可视化界面的提示一步一步安装，除了路径以外官网推荐默认选项，需要将路径添加到`PATH`路径下（理论上是自动的）。
3. 在弹出的命令行中运行`risk install`，但实际上并不一定需要你手动输入，可能直接就装起来了。然后选择安装选项，我的显示的是`3. MSYS2 and MINGW development tool chain`，会经历一个相对漫长的等待过程。
4. 打开新的命令行，为了让`PATH`的改变生效，不然找不到相关命令。运行`gem install jekyll bundler`，然后又是漫长的等待。
5. 最后装没装好，运行`jekyll -v`看一下版本就好。



#### 配置jekyll

1. 在拉取的`github.io`文件路径下运行：

   ```shell
   $ jekyll new --skip-bundle --force .
   ```

   由于当前路径下可能有文件，比如我的`README.md`，因此要强制执行一下。

2. 打开`Gemfile`文件，修改配置：

   - 注释掉`gem "jekyll"` 开头的行；

   - 编辑以 `# gem "github-pages"` 开头的行，以添加 `github-pages` gem。 将此行更改为：

     ```shell
     gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins
     ```

     将 `GITHUB-PAGES-VERSION` 替换为 `github-pages` gem 的最新支持版本。 可以在以下位置找到这个版本：https://pages.github.com/versions/。

3. 保存并关闭 Gemfile，从命令行中，运行 `bundle install`。



#### 本地测试

1. 运行`bundle install`

2. 安装`webrick`，新版本`Ruby`不再默认安装（踩坑了）：

   ```shell
   $ bundle add webrick
   ```

3. 在本地运行站点：

   ```shell
   $ bundle exec jekyll serve
   ```

4. 在http://localhost:4000访问并查看。





> **换源**：
>
> ```shell
> $ bundle config mirror.https://rubygems.org https://gems.ruby-china.com
> ```

