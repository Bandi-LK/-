
功能：将一个文件夹下，多个音乐文件按照不同专辑名称分类


```python
import pymediainfo
import os
import re
import shutil
```

pymediainfo模块的使用文档链接

https://buildmedia.readthedocs.org/media/pdf/pymediainfo/stable/pymediainfo.pdf

pymediainfo模块的安装方式

pip install pymediainfo


```python
'''
新建目录功能(自动忽略dir_name中的无效字符)
dir_path 输入目录路径
dir_name 输入文件
'''
def mkdir(dir_path, dir_name):

    # 删掉开头和结尾的空格
    dir_name = dir_name.strip()
    
    # 利用正则模块过滤掉windows系统非法字符
    dir_name = re.sub('[\/:*?"<>|]', '-', dir_name)
 
    # 判断路径是否存在
    dir_path = dir_path + dir_name
    is_exists = os.path.exists(dir_path)
 
    # 如果目录不存在，新建目录并返回True
    if not is_exists:
        os.makedirs(dir_path) 
        print(dir_path + " 目录创建成功") 
        return True
    
    # 如果目录存在，新建目录失败，提示已存在并返回FALSE
    print(dir_path + " 目录已存在") 
    return False
```


```python
'''
获取dir_path下所有媒体文件的路径及文件名,返回所有媒体文件的专辑
dir_path 输入目录路径
'''
def getAllMediaFileAlbum(dir_path):
    
    list_media_path = [];       # 存储本目录下媒体文件路径

    # 爬取整个目录下面的所有文件名称
    for list_root, list_dir, list_file in os.walk(dir_path):
        for file in list_file:
            # 将所有媒体文件路径记录到list_media_path中
            list_media_path.append(dir_path + file)


    list_media_file_dict = [];   # 存储所有媒体文件详细内容
    list_media_album = [];       # 存储所有媒体文件的专辑

    # 读取每个媒体文件的详细信息，分别记录详细信息和专辑
    for media_path in list_media_path:
        media_file = pymediainfo.MediaInfo.parse(media_path)
        list_media_file_dict.append(media_file.to_data())
        list_media_album.append(media_file.to_data()['tracks'][0]['album']) 
    
    # 返回媒体文件专辑
    return list_media_album
```


```python
'''
将media_file_dir_path下的媒体文件根据专辑名称放到对应的专辑文件夹下
media_file_dir_path 媒体文件路径 
result_dir_path 分类结果路径
'''
def moveFilesToAlbumDir(media_file_dir_path, classify_result_dir_path):
    
    # 遍历dir_path下所有的媒体文件，根据删除windows非法字符后的专辑名称，保存到对应的文件夹下
    
    # 爬取整个目录下面的所有文件名称
    for list_root, list_dir, list_file in os.walk(media_file_dir_path):
        for file in list_file:
            # 记录文件路径
            file_path = media_file_dir_path + file
            # 记录当前媒体文件信息
            media_file = pymediainfo.MediaInfo.parse(file_path)
            # 记录当前媒体文件专辑名称
            media_album_name = media_file.to_data()['tracks'][0]['album']
            # 得到当前媒体文件应该移动到的目录
            media_album_name = media_album_name.strip()
            media_album_dir_name = re.sub('[\/:*?"<>|]', '-', media_album_name)
            media_album_dir_path = classify_result_dir_path + media_album_dir_name
            # 移动文件
            shutil.move(file_path, media_album_dir_path)
```


```python
media_file_dir_path = "C:/Users/luobi/Desktop/wait/"   # 媒体文件路径 
classify_result_dir_path = "C:/Users/luobi/Desktop/dir_wait/" # 分类结果路径

# 获取dir_path下所有媒体文件的路径及文件名,返回所有媒体文件的专辑
list_media_album = getAllMediaFileAlbum(media_file_dir_path)


# 将所有的专辑名称创建文件夹
for media_album in list_media_album:
    mkdir(result_dir_path, media_album)
    
    
# 将每首音乐移动到对应专辑文件夹下
moveFilesToAlbumDir(media_file_dir_path, classify_result_dir_path)
```
