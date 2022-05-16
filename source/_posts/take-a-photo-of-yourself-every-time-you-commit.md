title: 提交代码时来张自拍
date: 2015-11-26 22:56:36
tags:
- Git
- 翻译
---
首先安装**imagesnap**，[https://github.com/alexwilliamsca/imagesnap](https://github.com/alexwilliamsca/imagesnap)或者Mac系统下通过homebrew安装：

```shell
brew install imagesnap
```

新建`~/.gitshots`目录：

```
mkdir ~/.gitshots
```

添加下面的文件`post-commit`到你代码目录的git hooks中：
```
#!/usr/bin/env ruby
file="~/.gitshots/#{Time.now.to_i}.jpg"
unless File.directory?(File.expand_path("../../rebase-merge", __FILE__))
  puts "Taking capture into #{file}!"
  system "imagesnap -q -w 3 #{file} &"
end
exit 0
```

为这个文件添加可执行权限：
```
chmod +x .git/hooks/post-commit
```

> 钩子(hooks)是一些在"$GIT-DIR/hooks"目录的脚本, 在被特定的事件(certain points)触发后被调用。当"git init"命令被调用后, 一些非常有用的示例钩子文件(hooks)被拷到新仓库的hooks目录中; 但是在默认情况下这些钩子(hooks)是不生效的。 把这些钩子文件(hooks)的".sample"文件名后缀去掉就可以使它们生效了。

这样就可以了！😎

----------

你还可以把这些图片合成为视频：[http://www.dayofthenewdan.com/projects/tlassemble](http://www.dayofthenewdan.com/projects/tlassemble)

我录的视频样例：[http://7d9pyw.com1.z0.glb.clouddn.com/2015-11-26T23:49:57.198500_test.mov](http://7d9pyw.com1.z0.glb.clouddn.com/2015-11-26T23:49:57.198500_test.mov)

原文链接：[Take a photo of yourself every time you commit](https://coderwall.com/p/xlatfq/take-a-photo-of-yourself-every-time-you-commit?p=1&q=)

GitHook参考：[http://gitbook.liuhui998.com/5_8.html](http://gitbook.liuhui998.com/5_8.html)
