import asyncio

import aiohttp

import aiofiles

import os

from Crypto.Cipher import AES

import re

import shutil





class File_create:

    def __init__(self, file_path, vedio_name):
        self.file_path = file_path
        self.vedio_name = vedio_name

    def create(self):
        os.system(rf'md {self.file_path}\{self.vedio_name}\ts')
        os.system(rf'md {self.file_path}\{self.vedio_name}\decode_ts')


class Async:

    def __init__(self, m3u8_url, file_path, vedio_name):
        self.m3u8_url = m3u8_url
        self.file_path = file_path
        self.vedio_name = vedio_name

    async def decode_one_ts(self, aes, name):
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\ts\{name}', 'rb') as fp1:
            data = aes.decrypt(await fp1.read())
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\decode_ts\{name}', 'wb') as fp2:
            await fp2.write(data)
        print(name + '解密完毕!')

    async def decode_all_ts(self, key):
        aes = AES.new(key=key, IV=b'0000000000000000', mode=AES.MODE_CBC)
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\ts文件的名称.text', 'r') as fp:
            tasks = []
            async for name in fp:
                name = name.strip('\n')
                tasks.append(asyncio.create_task(self.decode_one_ts(aes, name)))
            await asyncio.wait(tasks)

    async def get_key(self, session, key_url):
        async with session.get(url=key_url) as resp:
            key = await resp.read()
        return key

    async def get_data(self, session, ts_url):
        try:
            async with session.get(url=ts_url) as resp:
                data = await resp.content.read()
                if len(data) == 0:
                    return 'no', None
                return 'yes', data
        except asyncio.exceptions.TimeoutError:
            return 'no', None
        except aiohttp.client_exceptions.ServerDisconnectedError:
            return 'no', None
        except aiohttp.client_exceptions.ClientOSError:
            return 'no', None

    async def one_ts_download(self, session, ts_url, name):
        while True:
            judge, data = await self.get_data(session, ts_url)
            if judge == 'yes':
                break
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\ts\{name}', 'wb') as fp:
            await fp.write(data)
        print(name + '下载完毕!')

    async def download_all_ts(self, session, ts_urls):
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\ts文件的名称.text', 'w') as fp:
            tasks = []
            num = 0
            for ts_url in ts_urls:
                name = '{:0>5}.ts'.format(num)
                await fp.write(name + '\n')
                tasks.append(asyncio.create_task(self.one_ts_download(session, ts_url, name)))
                num += 1
            await asyncio.wait(tasks)

    async def url_compose(self, url_header, uncomplete_url):
        result_url_list = []
        uncomplete_url_list = uncomplete_url.split('/')
        for item in uncomplete_url_list:
            if item not in url_header:
                result_url_list.append(item)
        complete_url = url_header + '/'.join(result_url_list)
        return complete_url

    async def all_urls_get(self, session):
        async with session.get(self.m3u8_url) as resp:
            m3u8_content = await resp.read()
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\{self.vedio_name}.m3u8', 'wb') as fp:
            await fp.write(m3u8_content)

        url_header = self.m3u8_url.strip('index.m3u8')
        key_url = ''
        ts_urls = []
        async with aiofiles.open(rf'{self.file_path}\{self.vedio_name}\{self.vedio_name}.m3u8', 'r') as fp:
            async for item in fp:
                # 将key的url提取出来
                if 'key' in item:
                    key_url = re.compile(r'URI="(?P<key_url>.*?)"', re.S).search(item).group('key_url')
                    if 'http' not in key_url:
                        key_url = await self.url_compose(url_header, key_url)

                # 将文件里的所有ts文件的地址都提取出来
                if '#' not in item:
                    ts_url = item.strip()
                    if 'http' not in ts_url:
                        ts_url = await self.url_compose(url_header, ts_url)
                    ts_urls.append(ts_url)

        return key_url, ts_urls

    async def async_main(self):
        # 设置超时时间,默认为30秒
        timeout = aiohttp.ClientTimeout(total=30)
        async with aiohttp.ClientSession(timeout=timeout) as session:
            key_url, ts_urls = await self.all_urls_get(session)
            # print(key_url,ts_urls)
            await self.download_all_ts(session, ts_urls)
        if key_url == '':
            os.system(
                rf'copy /b {self.file_path}\{self.vedio_name}\ts\*.ts {self.file_path}\{self.vedio_name}.mp4')
        else:
            try:
                key = await self.get_key(session, key_url)
            except RuntimeError:
                async with aiohttp.ClientSession() as session1:
                    key = await self.get_key(session1, key_url)
            await self.decode_all_ts(key)
            os.system(
                rf'copy /b {self.file_path}\{self.vedio_name}\decode_ts\*.ts {self.file_path}.mp4')




class MainProcess:

    '''用户需要依次给出存储文件夹地址,存储的该视频的名称,相应的m3u8文件地址'''

    def __init__(self, file_path, video_name, m3u8_url):
        self.file_path = file_path
        self.video_name = video_name
        self.m3u8_url = m3u8_url

    def __init_async_get_video(self):
        file_create = File_create(self.file_path, self.video_name)
        file_create.create()
        print('相应的文件夹已经创造完毕!')
        return self.file_path, self.video_name

    def __get_mp4_video(self):
        boss = Async(self.m3u8_url, self.file_path, self.video_name)
        asyncio.get_event_loop().run_until_complete(boss.async_main())
        print(self.video_name + '下载完毕!')

        # 清理当前下载的文件,为后续文件的下载做准备
        shutil.rmtree(rf'{self.file_path}\{self.video_name}')

    def get_video(self):
        self.__init_async_get_video()
        self.__get_mp4_video()

