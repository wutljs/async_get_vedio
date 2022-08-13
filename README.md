# async_get_vedio

首先确保您已经正确安装了python3.9或更新版本的python编译环境。

1.作者在Pypi上传了一个叫做async_get_vedio的包，此文档是其使用的说明文档。具体文件实现方法请查看另外两个branches。

2.async_get_vedio依赖于python一些开源库，所以请确保您已经已经正确下载好如下的第三方库：aiofiles,aiohttp,Crypto,pyautogui。

3.下载这个包的方法为: pip install async_get_cedio

4.使用方法:用户先调用async_get_vedio. get_vedio. creat_files()完成文件夹的创建,该函数会有两个返回值,为file_path和vedio_name;再调用async_get_vedio. get_vedio.get_mp4_vedio(file_path,vedio_name)即可下载一个. mp4文件。

5.建议用户在调用完async_get_vedio. get_vedio.get_mp4_vedio(file_path,vedio_name)后，立刻调用async_get_vedio. get_vedio.clear_useless_files(file_path,vedio_name)，以确保下一个.mp4文件能够正常下载和不会占用过多的空间去存放没有意义的ts视频。

6.由于pyautogui. typewrite在某种情况时不幸不支持中文，建议输入相关信息时不要输入中文名称。否则async_get_vedio. get_vedio.clear_useless_files(file_path,vedio_name)可能工作失败。

7.根据提示，您将输入一个.m3u8文件的下载网址。注意：根据这个网址下载的.m3u8文件里面应该含有全部.ts(或者.jpg,.png)文件的下载相关路径。按照提示全部输入相关信息后，您会在您指定的文件夹里面发现您想要的.mp4文件。
