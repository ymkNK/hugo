---
title: 从Jekyll迁移至Hugo
author: ymkNK
categories: [Tech]
tags: [Ruby,Tech,Hugo]
date: 2021-05-27 19:52:17 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/26.jpeg
slug: 2021/5/rake-file-usage
toc: true
---
最近将博客背后的静态引擎，从Jekyll框架切换为了Hugo。也折腾了不少时间，现在总结一下其中的过程吧

### 准备
1. Go语言的环境 [Go官网](https://golang.org/)
2. Hugo的安装 [Hugo官网](https://gohugo.io/)
3. 选择一个喜欢的hugo主题进行搭建, [Hugo官网主题](https://themes.gohugo.io/)（本博客使用的主题是[hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack)）

### 主题个性化配置
hugo创建的基本流程网络上很多，这里就不过多赘述，一般选择了相应主题后，相关的主题介绍页，都会介绍如何使用，按照主题页指引即可。

这里想主要讲一下主题的适配，hugo的主题样式主要是由layouts这个文件夹下的样式作为模板，themes下所使用的模板中的layouts也会生效，因此我们可以利用这个特性，在站点项目下面的layouts重写同名的布局文件。
（本博客的gitalk就是利用了这一特性对主题进行了个性化配置）。

### 撰写文章
不同的hugo主题可能会有不同的文章存放策略，但是一定都是放在contents目录下，本文是放在contents/post目录下

markdown文件的头格式如下

    ---
    title: 从Jekyll迁移至Hugo
    author: ymkNK
    categories: [Tech]
    tags: [Ruby,Tech,Hugo]
    date: 2021-05-27 19:52:17 +0800
    img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/26.jpeg
    slug: 2021/5/rake-file-usage
    ---
    
用`---`和`+++`包围都可以，只是内容的格式要求有所区别，前者使用yaml的格式，后者使用toml的格式

### Rakefile
可以使用hugo new 快速创建文章，但是如果像我这样有定制化需求的创建（比如本博客的文章图片，以及目录格式等等），就需要手动进行处理，因此我使用了rakefile来通过简单的命令快速创建生成文章以及Tag

#### rake new
快速创建文章，并且可以随机生成oss图片库中的随机一个图片链接作为封面，按照创建的时间，自动放在content/post/#{@year}/#{@month}的目录下，方便归档（后期有考虑同时快速创建tags与categories）

#### rake git
快速执行git，一键提交本次更改的内容

#### rake hugo
快速构建生成hugo的静态网页，public文件夹下就是站点项目，本博客托管在github，因此可以直接push到仓库，进行直接发布

#### 源代码
```Ruby
task :default => :new

require 'fileutils'


desc "创建新 post"
task :new do
    time = Time.new

    puts "请输入要创建的 文件名字："
    @name = STDIN.gets.chomp
    @name = @name.downcase.strip.gsub(' ', '-')

    puts "请输入文章的 标题："
    @title = STDIN.gets.chomp

    puts "请输入文章的 categories, 以逗号分隔："
    @categories = STDIN.gets.chomp
    @categories = format_tags_or_categories @origin = @categories

    puts "请输入文章的 tags, 以逗号分隔："
    @tags = STDIN.gets.chomp
    @tags = format_tags_or_categories @origin = @tags

    # 需要检查一下文章目录是否存在
    @root_path = "content/post"

    check_or_create_dir(@dir = "#{@root_path}/#{time.year}")
    check_or_create_dir(@dir = "#{@root_path}/#{time.year}/#{time.month}")

    @date = Time.now.strftime("%F")
    @base_path = "#{time.year}/#{time.month}"
    @file_name = @date + "-" + @name

    @slug = "#{@base_path}/#{@name}"
    @slug = @slug.downcase.strip.gsub(' ', '-')

    # 生成随机封面
    @img_num = rand(1...31)
    @directory_url = "#{@root_path}/#{@base_path}"

    # 对应分类的文件放进对应的文件夹当中
    @post_name = "#{@directory_url}/#{@file_name}.md"

    @content = "---" + "\n" +
               "title: #{@title}" + "\n" +
               "author: ymkNK" + "\n" +
               "categories: [#{@categories}]" + "\n" +
               "tags: [#{@tags}]" + "\n" +
               "date: #{Time.now}" + "\n" +
               "img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/#{@img_num}.jpeg" + "\n" +
               "slug: #{@slug}" + "\n" +
               "---"
    create_file @file_dir = @post_name, @content

end

task :git do
  puts "请输入git内容 m"
    @msg = STDIN.gets.chomp

    @slug = "#{@msg}"
    @slug = @slug.downcase.strip.gsub(' ', '-')
    @date = Time.now.strftime("%F")
    @finalmsg = "#{@date}-#{@slug}"
    system "git add ."
    system "git commit -m \"#{@finalmsg}\""
    system "git status"
    system "git push"
    # system "git push github"
    # system "git push gitee"
    puts "git push \"#{@finalmsg}\""
end

task :hugo do
    @slug = 'build hugo'
    @date = Time.now.strftime("%F")
    @finalmsg = "#{@date}-#{@slug}"
    system "hugo"
    system "cd public && git add . && git commit -m \"#{@finalmsg}\" && git status && git push"
    puts "git push \"#{@finalmsg}\""
end

task :tag do
    puts "请输入要创建的tag名字："
    @tag = STDIN.gets.chomp
    @type = "tags"
    build_tag_or_category @type, @tag
end

task :category do
    puts "请输入要创建的category名字："
    @tag = STDIN.gets.chomp
    @type = "categories"
    build_tag_or_category @type, @tag
end

####################### 方法区 #######################

def format_tags_or_categories(origin)
    split_array = @origin.split(",")
    res = Array.new
    res = split_array.collect{ |x|x.capitalize }
    return res.join(",")
end

def check_or_create_dir(dir)
    puts "Check or create the dir : #{@dir}"
    if File.directory?(@dir)
        puts "The directory: #{@dir} has existed."
    else
        FileUtils.mkdir(@dir)
        puts "The directory: #{@dir} has created."
    end
end

def build_tag_or_category (type, tag)
    @tag = tag.capitalize
    @directory_name = @tag.downcase
    @slug = "#{@directory_name}"
    @img = "https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/tags/#{@directory_name}.jpg"

    @directory_url = "content/#{@type}/#{@directory_name}"
    create_tag_or_category_file @directory_url, @tag, @slug, @img
    puts "rake new #{@type} #{@tag} end."
end

def create_tag_or_category_file (directory_url, tag, slug, img)
    check_or_create_dir @dir=@directory_url

    # 对应分类的文件放进对应的categories或者tags文件夹当中
    @file_dir = "#{@directory_url}/_index.md"

    @content = "---" + "\n" +
               "title: #{@tag}" + "\n" +
               "description: #{@tag}" + "\n" +
               "slug: #{@slug}" + "\n" +
               "img: #{@img}" + "\n" +
               "style:" + "\n" +
               "    background: #2a9d8f" + "\n" +
               "    color: #fff" + "\n" +
               "---"
    create_file @file_dir, @content
end

def create_file (file_dir, content)
    if File.exist?(@file_dir)
        puts "#{@file_dir} 文件已经存在！创建失败"
        return
    end

    FileUtils.touch(@file_dir)
    open(@file_dir, 'a') do |file|
        file.puts @content
    end
    puts "#{@file_dir} 文件创建成功 内容如下:"
    puts @content
end

```
