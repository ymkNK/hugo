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
               "img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/#{@img_num}.jpg" + "\n" +
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
