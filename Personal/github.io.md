## 在GitHub上搭建个人博客

- https://docs.github.com/zh/pages/quickstart
- https://stackoverflow.com/questions/37405528/installing-ruby-2-3-on-wsl-windows-subsystem-for-linux



### 简明步骤笔记



#### 在Windows上安装jekyll

1. 在Windows上安装`jekyll`，前往https://jekyllrb.com/docs/installation/windows/找到安装包，下载`Ruby`和`Devkit`作为Windows上的依托https://rubyinstaller.org/downloads/。
2. 根据可视化界面的提示一步一步安装，除了路径以外官网推荐默认选项，需要将路径添加到`PATH`路径下（理论上是自动的）。
3. 在弹出的命令行中运行`risk install`，但实际上并不一定需要你手动输入，可能直接就装起来了。然后选择安装选项，我的显示的是`3. MSYS2 and MINGW development tool chain`，会经历一个相对漫长的等待过程。
4. 打开新的命令行，为了让`PATH`的改变生效，不然找不到相关命令。运行`gem install jekyll bundler`，然后又是漫长的等待。
5. 最后装没装好，运行`jekyll -v`看一下版本就好。



#### 在WSL2上安装jekyll

- 2023-12-09 UPDATE

1. 安装`Ruby`，首先安装一些必要的包：

   ```shell
   $ sudo apt-get update
   $ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev
   ```

2. windows环境下用了`RubyInstaller`，这次用一下三方的安装工具和包管理器`rbenv`：

   ```shell
   cd
   git clone https://github.com/rbenv/rbenv.git ~/.rbenv
   echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
   echo 'eval "$(rbenv init -)"' >> ~/.bashrc
   exec $SHELL
   
   git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
   echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
   exec $SHELL
   
   rbenv install 3.2.2
   rbenv global 3.2.2
   ruby -v
   ```

   经过一定时间的等待：

   ```shell
   ruby 3.2.2 (2023-03-30 revision e51014f9c0) [x86_64-linux]
   ```



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

