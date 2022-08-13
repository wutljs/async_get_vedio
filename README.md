# async_get_vedio:get_vedio.py

from async_get_vedio import aiofiles

from async_get_vedio import aiohttp

from async_get_vedio import asyncio

from async_get_vedio import AES

from async_get_vedio import os

from async_get_vedio import re

from async_get_vedio import pyautogui as pyg

class File_creat:

    def __init__(self, file_path, vedio_name):
        self.file_path = file_path
        self.vedio_name = vedio_name

    def creat(self):
        os.system(rf'md {self.file_path}\{self.vedio_name}')
        os.system(rf'md {self.file_path}\{self.vedio_name}\ts')
        os.system(rf'md {self.file_path}\{self.vedio_name}\decode_ts')
        return None

    def user_suggestions(self):
        suggestion = f'使用这个包前,请确保已经安装好aiohttp,aiofiles,Crypto库.这个包是利用这些库进行进一步分析.\n' \
                     '第一次编写包,可能功能不太完善.作者由衷的希望各位可以与我交流,欢迎指出我的不足.'
        with open(rf'{self.file_path}\{self.vedio_name}\suggestion.txt', 'w', encoding='utf-8') as fp:
            fp.write(suggestion)
        return None

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
                if num < 10:
                    name = f'000{num}.ts'
                elif 10 <= num < 100:
                    name = f'00{num}.ts'
                elif 100 <= num < 1000:
                    name = f'0{num}.ts'
                else:
                    name = f'{num}.ts'
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
                if 'key' in item:
                    key_url = re.compile(r'URI="(?P<key_url>.*?)"', re.S).search(item).group('key_url')
                    if 'http' not in key_url:
                        key_url = await self.url_compose(url_header, key_url)

                if '#' not in item:
                    ts_url = item.strip()
                    if 'http' not in ts_url:
                        ts_url = await self.url_compose(url_header, ts_url)
                    ts_urls.append(ts_url)

        return key_url, ts_urls

    async def async_main(self):
        timeout = aiohttp.ClientTimeout(total=60)
        async with aiohttp.ClientSession(timeout=timeout) as session:
            key_url, ts_urls = await self.all_urls_get(session)
            await self.download_all_ts(session, ts_urls)
        if key_url == '':
            os.system(
                rf'copy /b {self.file_path}\{self.vedio_name}\ts\*.ts {self.file_path}\{self.vedio_name}\{self.vedio_name}.mp4')
        else:
            try:
                key = await self.get_key(session, key_url)
            except RuntimeError:
                async with aiohttp.ClientSession() as session1:
                    key = await self.get_key(session1, key_url)
            await self.decode_all_ts(key)
            os.system(
                rf'copy /b {self.file_path}\{self.vedio_name}\decode_ts\*.ts {self.file_path}\{self.vedio_name}\{self.vedio_name}.mp4')

def main_step_one():

    file_path = input(r'请输入一个文件夹地址来存储视频文件,如 C:\Users\admin\xxx:')
    
    vedio_name = input('请输入该视频的名字:')
    
    file_creat = File_creat(file_path, vedio_name)
    
    file_creat.creat()
    
    file_creat.user_suggestions()
    
    print('相应的文件夹已经创造完毕!')
    
    return file_path, vedio_name

def main_step_two(file_path, vedio_name):

    m3u8_url = input('请输入一个m3u8文件的下载网址,注意:\n该m3u8文件里应含有所有的ts文件的下载网址.\n')
    
    boss = Async(m3u8_url, file_path, vedio_name)
    
    asyncio.get_event_loop().run_until_complete(boss.async_main())
    
    print(vedio_name + '下载完毕,请观看!')

def main_step_three(file_path, vedio_name):

    del_file_path_list = [rf'{file_path}\{vedio_name}\ts', rf'{file_path}\{vedio_name}\decode_ts']

    pyg.hotkey('win', 'r')
    
    pyg.typewrite('cmd')
    
    pyg.hotkey('enter')
    
    pyg.hotkey('enter')

    for del_file_path in del_file_path_list:
    
        pyg.typewrite(f'del {del_file_path}')
        
        pyg.hotkey('enter')
        
        pyg.typewrite('Y')
        
        pyg.hotkey('enter')
        
    os.remove(rf'{file_path}\{vedio_name}\ts文件的名称.text')
    
    os.remove(rf'{file_path}\{vedio_name}\{vedio_name}.m3u8')

    pyg.typewrite('exit')
    
    pyg.hotkey('enter')

def main():

    file_path, vedio_name = main_step_one()
    
    main_step_two(file_path, vedio_name)
    
    main_step_three(file_path,vedio_name)
