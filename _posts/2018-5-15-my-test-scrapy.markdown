---
layout: post
title: "一次滑动验证码的爬虫"
img: bowen42.png
date: 2018-05-15 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python]
---

**徐静**

最近有人问我关于爬虫中滑块验证码的识别问题，然后尝试了一下，主要分为以下几步：

+ 步骤一:点击按钮，弹出没有缺口的图片
+ 步骤二：获取步骤一的图片
+ 步骤三：点击滑动按钮，弹出带缺口的图片
+ 步骤四：获取带缺口的图片
+ 步骤五：对比两张图片的所有RBG像素点，得到不一样像素点的x值，即要移动的距离
+ 步骤六：模拟人的行为习惯（先匀加速拖动后匀减速拖动），把需要拖动的总距离分成一段一段小的轨迹
+ 步骤七：按照轨迹拖动，完全验证
+ 步骤八：完成登录

上代码：

```python

base_url = "http://www.geetest.com/type"


from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from PIL import Image
import time
import numpy as np

def get_snap(driver,names):
    driver.save_screenshot(names)
    page_snap_obj=Image.open(names)
    return page_snap_obj

def get_image(driver,names):
    img=driver.find_element_by_class_name('geetest_canvas_img')
    time.sleep(2)
    location=img.location
    size=img.size

    left=location['x']
    top=location['y']
    right=left+size['width']
    bottom=top+size['height']

    page_snap_obj=get_snap(driver,names)
    image_obj=page_snap_obj.crop((left,top,right,bottom))
    #image_obj.show()
    return image_obj

def get_distance(image1,image2):
    start=int(np.ceil(image2.size[0]/4))
    threhold=120

    for i in range(start,image1.size[0]):
        for j in range(image1.size[1]):
            rgb1=image1.load()[i,j]
            rgb2=image2.load()[i,j]
            res1=abs(rgb1[0]-rgb2[0])
            res2=abs(rgb1[1]-rgb2[1])
            res3=abs(rgb1[2]-rgb2[2])
            #print(res1,res2,res3)
            if not (res1 < threhold and res2 < threhold and res3 < threhold):
                return i - 4
    return i - 4

def get_tracks(distance):
    '''
    本质来源于物理学中的加速度算距离： s = vt + 1/2 at^2
                                    v = v_0 + at

    在这里：总距离S= distance+20
            加速度：前3/5S加速度2，后半部分加速度是-3

    '''
    distance+=20 #先滑过一点，最后再反着滑动回来
    v=0
    t=0.2
    forward_tracks=[]

    current=0
    mid=distance*3/5
    while current < distance:
        if current < mid:
            a=2
        else:
            a=-3

        s=v*t+0.5*a*(t**2)
        v=v+a*t
        current+=s
        forward_tracks.append(round(s))

    #反着滑动到准确位置
    back_tracks=[-3,-3,-3,-2,-2,-1,-2,-1,-1,-1] #总共等于-10

    return {'forward_tracks':list(forward_tracks),'back_tracks':back_tracks}


#判断元素是否存在
#'geetest_success_radar_tip'
def isElementExist(driver,element):
    flag=True
    browser=driver
    try:
        browser.find_element_by_class_name(element)
        return flag
    
    except:
        flag=False
        return flag


def my_scrapy():
    try:
        # 1、输入账号密码回车
        driver = webdriver.Chrome()
        driver.implicitly_wait(3)
        driver.get("http://www.geetest.com/type")


        time.sleep(1)
        driver.find_element_by_xpath('//*[@id="app"]/section/div/ul/li[2]/h2').click()
        #1获取全相
        time.sleep(0.5)
        driver.find_element_by_class_name('geetest_wait').click()
        time.sleep(1)
        image1 = get_image(driver,'before.png')

        #2获取有缺口的图像
        driver.find_element_by_xpath('/html/body/div[3]/div[2]/div[2]/div[1]/div[2]/div[2]').click()
        image2 = get_image(driver,'after.png')

        # 3对比两种图片的像素点，找出位移
        distance = get_distance(image1, image2)

        # 4模拟人的行为习惯，根据总位移得到行为轨迹
        tracks = get_tracks(distance)
        #print(tracks)

        # 5按照行动轨迹先正向滑动，后反滑动
        button = driver.find_element_by_class_name('geetest_slider_button')
        ActionChains(driver).click_and_hold(button).perform()

        #6正常滑动
        for track in tracks['forward_tracks']:
            ActionChains(driver).move_by_offset(xoffset=track, yoffset=0).perform()

        # 7返回
        time.sleep(0.5)
        for back_track in tracks['back_tracks']:
            ActionChains(driver).move_by_offset(xoffset=back_track, yoffset=0).perform()

        # 8小幅晃动模拟人操作
        ActionChains(driver).move_by_offset(xoffset=-3, yoffset=0).perform()
        ActionChains(driver).move_by_offset(xoffset=3, yoffset=0).perform()

        ActionChains(driver).move_by_offset(xoffset=2, yoffset=0).perform()
        ActionChains(driver).move_by_offset(xoffset=-2, yoffset=0).perform()

        # 9松开滑块
        time.sleep(0.5)
        ActionChains(driver).release().perform()

        #10记录本次是否验证成功
        time.sleep(3)
        if driver.find_element_by_class_name('geetest_success_radar_tip').text=='':
            success_tag = False
        else:
            success_tag = True
        return success_tag
	    
    finally:
	    driver.close()



if __name__=='__main__':

    my_success = 0
    for i in range(0,500):
        my_test = my_scrapy()
        print('[+]第'+str(i+1)+r'/500次模拟状态:'+str(my_test))
        if my_test:
            my_success += 1

    print(my_success)


```


