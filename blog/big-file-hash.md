# 计算超大文件md5值，实现类似各大网盘中的“秒传”功能算法

最近想做一个文件管理库，发现文件多到一定程度后只用文件名来管理有些困难，想尝试下用其它方法来管理文件，这时想起来各类网盘工具中的“秒传”功能，在传文件时，几个G的文件一下就传上去了，但其实就没传，原理据说就是计算文件的md5值，然后检查系统里已经有这个文件了，就不用在上传了，给用户显示“你的文件已经秒传成功”。

另外日常使用的git在提交文件时也有类似的操作，但用的不是md5而是sha1，打开“.git/objects/ ”目录里面就是sha1算法生成的文件。

sha1和md5的作用是相同的，但sha1碰撞难度更高，有更好的安全性，但在一般情况下，md5就已经够用了。

动手开始写代码，代码运行起来后，遇到了第一个困难，"磁盘占用率太高"，记得以前用机械硬盘时，windows动不动就磁盘占用率100%，用了各种办法都没法解决，以为这是微软在提示用户，你的硬盘应该升级了，最后只好升级固态硬盘解决了这个问题，没想到今天又遇到了这个问题。

我用的是php的md5_file()函数计算文件的md5值，当计算2G以上mp4文件的md5值时cpu 内存都正常，磁盘占用却率升到70%以上。于是我更换了编程语言，看是否还有这个问题。

首页找到了一个python计算文件md5值的封装，但测试下来也是一样的问题，磁盘占用率太高，于是又用上了js，在github找到了spark-md5.js这个库，在浏览器前端计算文件的md5值，发现虽然没有磁盘占用率太高的问题，但浏览器的内存占用又太高了。

于是就想能不能不判断下文件大小，对太大的文件不计算md5了，结合文件大小，最后修改时间，创建时间等因素来做文件的唯一标识（找出文件有多少个副本），这个方法并不好，因为文件的元信息是可以随意改动的，要造出两个元信息完全一致的文件并不困难。

经人指点后，感觉还是要用md5才行，但对于太大的文件，可以并不计算整个文件的md5，而是抽取文件部分内容计算hash。因为并不需要文件的全部内容。

对于太大的文件使用：文件大小+内容抽样(头、中、尾或每隔xx字节抽样一次哈希),写了一段php代码来验证这一思路，发现可行。测试下来没有磁盘占用率太高的情况，也不吃内存，没有性能问题，

``` php
function file_md5_16k($path){
  $size=filesize($path);//取得文件大小
  if($size>16384){//如果文件大于16kb
    $str=$size;
    $str.=file_get_contents($path,null,null,0,4096);#文件头部4kb
    $str.=file_get_contents($path,null,null,(($size/2)-2048),4096);#文件中部4kb
    $str.=file_get_contents($path,null,null,($size-4096),4096);#文件尾部4kb
    return md5($str);
  }else{ //文件不太，不抽样，直接计算整个文件的hash
    return md5_file($path);
  }
}
```
这里只是测试16kb以上的文件就用了抽样计算，实际应用中16k的文件太小了，大于16kb的文件太多，中间修改了一些内容很会产生重复。应该设置的更大一些。

md5存在重复可能，在md5基础上再结合文件类型，文件元信息等就可以对文件做唯一标识，避免文件重复，从而建立文件指纹库。

当然要实现文件“秒传”要做的远远不止这些，这个只是实现原理算法。

最后感谢emacs-china论坛的的热心用户 @twlz0ne @Liutos @SuperMMX 给予的指点。
