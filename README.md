# async_get_vedio

首先确保您已经正确安装了python3.8或者3.8以上版本的python编译环境。

1.作者在Pypi上传了一个叫做async_get_vedio的包，此文档是其使用的说明文档。具体文件实现方法请查看另外两个branches。

2.async_get_vedio依赖于python一些开源库，所以请确保您已经已经正确下载好如下的第三方库：aiofiles,aiohttp,Crypto。

3.下载这个包的方法为: pip install async_get_vedio

4.引入方法： 

from async_get_vedio.get_vedio import MainProcess

#实例化MainProcess类,传递参数:file_path,video_name,m3u8_url

mainprocess = MainProcess ( file_path, video_name, m3u8_url )

#调用get_video方法进行目标视频下载

mainprocess.get_video()

6.注意事项：***您提供的m3u8_url需要满足：根据这个网址下载的.m3u8文件里面应该含有全部.ts(或者.jpg,.png)文件的下载相关路径。按照提示全部输入相关信息后，您会在您指定的文件夹里面发现您想要的.mp4文件!!!***

7.样品展示:百度网盘:链接(无提取码)：https://pan.baidu.com/s/1AjnsS5o-qqLIbD1Q7OR2EA?pwd=ys57 
