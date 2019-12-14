---
layout: post
tags: [Jekyll]
comments: true
---
执行`Jekyll`本地浏览器预览指令
```
 bundle exec jekyll serve
```
进入浏览器输入`127.0.0.1:4000`，可以正常浏览首页，但是点击文章链接，则会显示`404`页面，查看控制台显示错误的`log`，如下：
```
PS D:\work\github\test\_site> bundle exec jekyll serve
Configuration file: none
            Source: D:/work/github/test/_site
       Destination: D:/work/github/test/_site/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.972 seconds.
 Auto-regeneration: enabled for 'D:/work/github/test/_site'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
[2019-12-14 09:20:28] ERROR `/2019-11-06-STM32-TIM1高级定时器配置快速入门/' not found.
[2019-12-14 09:20:37] ERROR `/2019-11-05-STM32-时钟树配置快速入门/' not found.
[2019-12-14 09:20:42] ERROR `/2019-11-04-STM32-J-scope的使用/' not found.
[2019-12-14 09:20:48] ERROR `/2019-10-28-Matlab-simulink-步进电机驱动仿真/' not found.
[2019-12-14 09:21:07] ERROR `/2019-10-25-STM32-正交解码功能采集编码器信号/' not found.
[2019-12-14 09:21:13] ERROR `/2019-10-24-STM32-互补PWM波形配置/' not found.
[2019-12-14 09:57:47] ERROR `/favicon.ico' not found.
[2019-12-14 10:07:20] ERROR `/2019-09-14-PMSM-一-电机的数学模型/' not found.
[2019-12-14 10:07:25] ERROR `/2019-09-20-STM32-标准库启动文件分析/' not found.
[2019-12-14 10:07:29] ERROR `/2019-10-25-STM32-正交解码功能采集编码器信号/' not found.
```

以上无法找到的文件，在`_site`文件夹下都存在，简单分析应该是编码格式的问题，导致`server`无法正确找到对应的文章；

**解决方案：**
找到文件`C:\Ruby26-x64\lib\ruby\2.6.0\webrick\httpservlet\filehandler.rb`，具体**安装目录根据实际情况确定**；

文件修改如下所示；
```
diff --git a/filehandler_back.rb b/filehandler.rb
index 601882e..10f51d6 100644
--- a/filehandler_back.rb
+++ b/filehandler.rb
@@ -283,6 +283,7 @@ module WEBrick
         # dirty hack for filesystem encoding; in nature, File.expand_path
         # should not be used for path normalization.  [Bug #3345]
         path = req.path_info.dup.force_encoding(Encoding.find("filesystem"))
+		path.force_encoding("UTF-8")
         if trailing_pathsep?(req.path_info)
           # File.expand_path removes the trailing path separator.
           # Adding a character is a workaround to save it.
@@ -330,6 +331,7 @@ module WEBrick
         path_info.unshift("")  # dummy for checking @root dir
         while base = path_info.first
           break if base == "/"
+		  base.force_encoding("UTF-8")
           break unless File.directory?(File.expand_path(res.filename + base))
           shift_path_info(req, res, path_info)
           call_callback(:DirectoryCallback, req, res)

```

修改完，重新执行`bundle exec jekyll serve`，进入浏览器发现已经可以解决这个问题。