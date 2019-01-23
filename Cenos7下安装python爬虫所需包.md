# Cenos7下安装python爬虫所需包

安装setuptools

```shell
wget https://bootstrap.pypa.io/ez_setup.py -O - | python	

unzip setuptools-33.1.1.zip 

cd setuptools-33.1.1

python2.7 setup.py build
```

安装pip

```shell
wget https://bootstrap.pypa.io/get-pip.py

python  get-pip.py
```

安装requests

```shell
pip install requests
or
yum install -y python-requests
```

安装bs4

```shell
python -m pip install bs4
```

安装beautifulsoup

```shell
wget https://www.crummy.com/software/BeautifulSoup/bs4/download/4.5/beautifulsoup4-4.5.1.tar.gz

tar -zxvf beautifulsoup4-4.5.1.tar.gz

cd beautifulsoup4-4.5.1

python2.7 setup.py install
```

安装lxml

```
pip install lxml
```

python2.x中分为 urllib,urllib2,urlparse等包

```python
import requests
from bs4 import BeautifulSoup
from urlparse import urlparse
from urlparse import parse_qs
import sys

urllib2.urlopen("http://www.baidu.com")
urlparse.urlparse("http://www.baidu.com")
```

python3.0中将其合为一个包urllib

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urlparse
from urllib.parse import parse_qs
import sys

urllib.request.urlopen("http://www.baidu.com")
urllib.parse.urlparse("http://www.baidu.com")
```

```python
import requests
from urllib.parse import urlparse,parse_qs
from bs4 import BeautifulSoup
import sys,json

def write_to_file(content):
    with open("tt.txt","a",encoding="utf8") as f:
        f.write(content["title"]+"\t"+content["url"]+"\t"+content["tid"]+"\t")
        f.write(content["author"]["user_name"]+"\t"+content["author"]["uid"]+"\t")
        f.write(json.dumps(content["content"],ensure_ascii=False)+"\t")
        for usr in content["comments"]:
            try:
                f.write(usr["tid"]+":"+usr["content"].split("]")[1]+":"+usr["user_info"]["user_name"]+":"+usr["user_info"]["uid"]+"#")
            except:
                f.write(usr["tid"]+":"+"None"+":"+usr["user_info"]["user_name"]+":"+usr["user_info"]["uid"]+"#")
        f.write("\n")

def get_url_content(url):
    response=requests.get(url)
    if response.status_code==200:
        if "抱歉，指定的主题不存在或已被删除或正在被审核" in response.content.decode("utf8"):
            return False
        else:
            return response.content.decode("utf8")
    return False

def get_user_list(soup_object):
    user_list=[]
    user_html=soup_object.select(".authi")
    for i in range(len(user_html)):
        if i%2==0:
            user_name=user_html[i].a.string
            uid=parse_qs(user_html[i].a["href"])["uid"][0]
            user_list.append({"user_name":user_name,"uid":uid})
    return user_list

def get_content_list(soup_object):
    content_list=[]
    content_html=soup_object.select(".t_f")
    for i in range(len(content_html)):
        possmessage=content_html[i]["id"]
        tid=possmessage.split("_")[1]
        content=content_html[i].string
        content_list.append({"tid":tid,"content":content})
    return content_list

def get_parse_post(html_text):
    soup=BeautifulSoup(html_text,"lxml")
    title=soup.title.string
    try:
        title=title.split("-")[0]
    except:
        pass
    url=soup.link["href"]
    parseurl=urlparse(url)
    tid=parse_qs(parseurl.query)["tid"][0]
    user_list=get_user_list(soup)
    content_list=get_content_list(soup)
    for i in range(len(content_list)):
        content_list[i]["user_info"]=user_list[i]
    post_parse_data={
        "title":title,
        "url":url,
        "tid":tid,
        "author":user_list[0],
        "content":content_list[0]["content"],
        "comments":content_list[1:]
    } 
    return post_parse_data

def main():
    base_url= "http://114.112.74.138/forum.php?mod=viewthread&tid="
    for i in range(5830,5835):
        content_data=get_url_content(base_url+str(i))
        if content_data!=False:
            parse_data=get_parse_post(content_data)
            write_to_file(parse_data)
            print("get post data tid:%s"%(i))
        else:
            print("tid:%s not found"%(i))
        
m
```

