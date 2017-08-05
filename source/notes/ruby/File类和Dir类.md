---
title: File 类和 Dir 类
type: notes
order: 7
category: Ruby
---

```
require "fileutils"
#FileUtils.mv可以实现跨文件系统、驱动器的文件移动
FileUtils.cp(from, to)
FileUtils.mv(from, to)
```

* File.rename
* File.delete
* File.unlink
* File.directory?(path)
* Dir.pwd
* Dir.chdir(dir)
* Dir.open(path)
* Dir#close
* dir.read 遍历读取最先打开的目录下的内容
* Dir.glob 可以使用shell通配符`*`和 `?`匹配文件名，以数组返回。
* Dir.mkdir(path)
* Dir.rmdir(path) 要删除的目录必须为空目录

```
def traverse(path)
  if File.directory?(path)
    dir = Dir.open(path)
    while name = dir.read
      next if name == "."
      next if name == ".."
      traverse(path + "/" + name)
    end
    dir.close
  else
    process_file(path)
  end
end

def process_file(path)
  puts path
end

traverse(ARGV[0])
```
