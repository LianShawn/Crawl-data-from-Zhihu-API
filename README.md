# Use selenium to get data from Weibo
This part is about downloading data from weibo platform

#packages
```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.keys import Keys
import matplotlib.pylab as plt
import sys
import json
import chardet
import time
from bs4 import BeautifulSoup
import re
import pandas as pd
import csv
import datetime
```

#prepare
```driver = webdriver.Chrome("/Users/lianaxiang/Desktop/Weibo_crawl/chromedriver")

driver.maximize_window()
driver.get('https://passport.weibo.cn/signin/login') #获取web site content
time.sleep(2)  #延迟，为了加载元素，否则照顾到元素，出现异常
input_name = driver.find_element(By.ID,'loginName')
input_name.clear()
input_name.send_keys('***')
input_name.send_keys(Keys.ENTER)

input_pass = driver.find_element(By.ID,'loginPassword')
input_pass.clear()
input_pass.send_keys('***')  #输入密码
time.sleep(2)
driver.find_element(By.ID,'loginAction').click()

time.sleep(2)
```
#get the direct comment
```
def crawl_content(url):
    
    driver.get(url)
    comment_number=[]
    comment_url=[]
    content=[]
    times=[]
    news_account=[]
    page=15
    while page <81:
        info = driver.find_elements_by_class_name('cc')
        for value in info:
            comment_number.append(value.text)
            comment_url.append(value.get_attribute('href'))
        
        infor = driver.find_elements_by_class_name('ctt')
        for value1 in infor:
            content.append(value1.text)
            
        info1=driver.find_elements_by_class_name('ct')
        for value2 in info1:
            times.append(value2.text)
            
        infor2=driver.find_elements_by_class_name('nk')
        for value3 in infor2:
            news_account.append(value3.text)
    
        content_frame={
            
            'content':content,
            'comment_number':comment_number,
            'comment_url':comment_url,
            'tweet_time':times,
            'tweet_account':news_account
        }

        content_result=pd.DataFrame(content_frame)
        time.sleep(2)
        content_result.to_csv("test3.csv",index=False,sep=',')
        print('成功抓到第'+ str(page))
        driver.find_element_by_xpath("//*[@id='pagelist']/form/div/a[1]").click()
        page=page+1
        time.sleep(2)
        
    return content_result
```
#practice
```
url='https://weibo.cn/search/mblog?hideSearchFrame=&keyword=%E6%97%A5%E6%9C%ACAPA&advancedfilter=1&starttime=20170112&endtime=20170201&sort=hot'
crawl_content(url)
```
#crawl the second level comments
```
def crawl_comment3(url):
    time.sleep(2)
    driver.get(url)
    comment_content=[]
    comment_time=[]

    page1=1
    
    try:
        info2 = driver.find_element_by_xpath('//*[@id="pagelist"]/form/div')
        s=r'\d+'
        pattern = re.compile(s)
        m = pattern.findall(info2.text)
        page_number=int(m[1])
        print(page_number)
        
    except Exception as e:
        
        return
        
    
    
    
    while page1<= page_number:
        
        info=driver.find_elements_by_class_name('ctt')
        for value in info:
            comment_content.append(value.text)
        info1=driver.find_elements_by_class_name('ct')
        for value1 in info1:
            comment_time.append(value1.text)
        comments_url=driver.current_url
        
        
        comment_frame={
            "comments_url":comments_url,
            "comment_content":comment_content,
            "comment_time":comment_time
            
        }
        
        comment_result=pd.DataFrame(comment_frame)
        
        
        print('成功抓到第'+ str(page1))
        page1=page1+1
        if page1 == 20:
            page1=page1+1
        
        try:
            input_page = driver.find_element_by_xpath('//*[@id="pagelist"]/form/div/input[2]')
            input_page.clear()
            input_page.send_keys(page1)  #输入頁碼
            time.sleep(3)
            driver.find_element_by_xpath('//*[@id="pagelist"]/form/div/input[3]').click()
            
        except Exception as e:
            comment_result.to_csv("comment4.csv",index=False,sep=',',mode='a')
            driver.back()
            input_page = driver.find_element_by_xpath('//*[@id="pagelist"]/form/div/input[2]')
            input_page.clear()
            page1+=1
            input_page.send_keys(page1)  #输入頁碼
            driver.find_element_by_xpath('//*[@id="pagelist"]/form/div/input[3]').click()
    
    comment_result.to_csv("comment4.csv",index=False,sep=',',mode='a')
 ```
 
