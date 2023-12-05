```python
import re
import time
import random
import requests
import urllib3
from concurrent.futures import ThreadPoolExecutor
# 多线程 ↑ 不熟练,没搞,有能力的可以自己加
# 关闭证书警告↓
urllib3.disable_warnings()
"""
由于这个pixiv关闭了AccountPostkey的获取,导致无法登录,初学不是很会
        模拟登录作废↓
        R18区域不开放(悲)
"""
# # 以下是模拟登录(已作废)
# headers = {'Referer': 'https://www.pixiv.net/', 'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)
# AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'}
#
#
# def get_postkey():
#     login_url = 'https://accounts.pixiv.net/login?return_to=https%3A%2F%2Fwww.pixiv.net%2F&lang=zh&source=pc&view_type=page'
#     response = requests.get(url=login_url, headers=headers, verify=False)
#     response.encoding = 'utf-8'
#     html = response.text
#
#
# pixiv_id = "********"  # 你的pixiv账号
# password = '******'  # 你的pixiv密码
# # 不知为何这个postkey在网页源代码找不到了
# post_key=get_postkey()
# return_to = 'https://www.pixiv.net/'
#
# session = requests.Session()
#
# form_data = {
#     'pixiv_id': pixiv_id,
#     'password': password,
#     'return_to': return_to,
#     'post_key':post_key
# }
# login_url1 = ('https://accounts.pixiv.net/login?return_to='
#               'https%3A%2F%2Fwww.pixiv.net%2F&lang=zh&source=pc&view_type=page')
# res = session.post(url=login_url1, headers=headers, data=form_data)
# # 至此模拟登录成功

headers = {'Referer': 'https://www.pixiv.net/', 'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                                                              'AppleWebKit/537.36 (KHTML, like Gecko) '
                                                              'Chrome/119.0.0.0 Safari/537.36',}
global flag
global count
def download(result,count,n):
    print("若有日期输出则获取成功,否则无法获取")
    print("\n")
    pic_name = re.compile('"https://i.pximg.net/img-original/img/(?P<date>.*?)_p0.(?P<type>.*?)"')
    str = pic_name.finditer(result.text)
    for item in str:
        dit = item.groupdict()
        var = dit['date']
        type = dit['type']
        print(var)
        the_url = f'https://i.pximg.net/img-original/img/{var}_p0.jpg'
        if result.status_code == 404 or result.status_code == 403:
            the_url = f'https://i.pximg.net/img-original/img/{var}_p0.png'
        finalresult = requests.post(the_url, headers=headers, verify=False, timeout=5)
        r = random.randint(1, 4)
        time.sleep(r)
        print(f"开始下载图片{count}")
        # ./代表从当前目录,image是当前目录的image文件夹,image{n}_{count}.{type}是图片的名称,其中{type}是文件类型(jpg/png),可根据需求自行修改
        with open(f'./image/image{n}_{count}.{type}', mode='wb') as f:
            f.write(finalresult.content)
            print(f"图片{count}已下载成功！！\n")


# n是个随机数,用来区分每次的下载,并给与不同前缀
n = random.randint(20230000001, 20239999999)
print("--------开始进行pixiv图片爬取--------\n由Sakura_Akira连夜制作,有不足还望大佬纠正\n很多图片下载后无法正常浏览,因为有反扒(大概)\n运行代码前请"
      "在当前目录新建image文件夹(想作修改看下载部分注释)\n(若为pycharm,"
      "需要再对该文件夹右键标记为排除)")
user_input = int(input("选择要选择的下载方式:\n1. 指定图片网址的标号(111******-113999999)(不一定存在)\n2. 随机下载 \n3. 下载日推\n"))
if user_input == 1:
    count = int(input("输入要下载的图片标号: "))
    url = f'http://www.pixiv.net/artworks/{count}'
    print(url)
    result = requests.post(url, headers=headers, verify=False, timeout=5)
    download(result,count,n)
elif user_input == 2:
    number = int(input("选择要下载的图片数: "))
    if number > 0:
        count = 1
        for l in range(number):
            r = random.randint(111111, 3999999)
            url = f'http://www.pixiv.net/artworks/11{r}'
            print(url)
            result = requests.post(url, headers=headers)
            download(result, count,n)
            count += 1
    else:
        print("输入错误")
elif user_input == 3:
    # 日榜首页
    url = "https://www.pixiv.net/ranking.php"
    father = "https://www.pixiv.net/"
    child_url_list = []
    resp = requests.get(url)

    # 获取图片详细信息页面链接
    obj = re.compile(r"<div class=\"ranking-image-item\"><a href=\"(?P<image_URL>.*?)\"class", re.S)
    result = obj.finditer(resp.text)

    for it in result:
        list = it.group('image_URL')
        child_url = father + list.strip("/")
        child_url_list.append(child_url)

    # 获取原图片
    obj1 = re.compile(r"\"original\":\"(?P<download_URL>.*?)\"")
    headers = {'Referer': 'https://www.pixiv.net/'}  # 由于pixiv设置了图片防盗链，故需请求头添加Referer，作用为告诉服务器【本次访问通过主页发起】，防止访问被拒绝
    requests.packages.urllib3.disable_warnings()
    cnt = 0
    for href in child_url_list:
        if cnt > 7:
            break
        final = requests.get(href)
        real_image_url = obj1.search(final.text)
        DL_URL = real_image_url.group('download_URL')
        path = str(DL_URL).split('/')[-1]
        print(f"图片{path}开始下载(第{cnt + 1}张)")
        pic = requests.get(DL_URL, headers=headers, verify=False, timeout=5)
        with open(f"./image/{path}", mode='wb') as f:
            f.write(pic.content)
            f.close()
            print(f"第{cnt + 1}图片下载成功!")
        cnt = cnt + 1  # 限制下载数
    print("全部下载完成")
else:
    print("输入错误")
```
