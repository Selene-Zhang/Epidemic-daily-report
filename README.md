# Epidemic-daily-report
Epidemic daily report for homework
import requests, os
from matplotlib import pyplot as plt
from fake_useragent import UserAgent
from selenium import webdriver
from lxml import etree
from time import sleep
from matplotlib import pyplot as plt
import random, datetime
import matplotlib
from matplotlib import font_manager
from pyecharts.faker import Faker
from pyecharts import options as opts
from pyecharts.charts import Map
import numpy as np

chrome = webdriver.Chrome()
url = "https://voice.baidu.com/act/newpneumonia/newpneumonia"
chrome.get(url)
chrome.find_element_by_xpath("//div[@id='nationTable']/div").click()
# sleep(5)  # 等下拉的全部展示，才能取得下拉数据，不能放在执行js之前
html = chrome.page_source
e = etree.HTML(html)
province_name_list = e.xpath("//div[@id='nationTable']//div[@class='VirusTable_1-1-271_AcDK7v']/span/text()")
print(province_name_list)
data_all = e.xpath("//div[@id='nationTable']/table/tbody/tr/td/text()")
# print(data_all,len(data_all))   # 170

province_data_list = []  # 每个元素是一个省的新增、现有、累计、治愈和死亡
add_list = []  # 新增
current_list = []  # 现有
total_list = []  # 累计
cure_list = []  # 治愈
die_list = []  # 治愈
for i in range(int(len(data_all) / 5)):
	#  新增   现有     累计   治愈  死亡
	# ['22', '228', '821', '584', '9']
	d = data_all[i * 5:(i + 1) * 5]
	province_data_list.append(d)
	if d[0] != '待公布':
		add_list.append(int(d[0]))
	else:
		add_list.append(0)
	if d[1] != '待公布':
		current_list.append(int(d[1]))
	else:
		current_list.append(0)
	if d[2] != '待公布':
		total_list.append(int(d[2]))
	else:
		total_list.append(0)
	if d[3] != '待公布':
		cure_list.append(int(d[3]))
	else:
		cure_list.append(0)
	if d[4] != '待公布':
		die_list.append(int(d[4]))
	else:
		die_list.append(0)


def is_exist(paths):
	for path in paths:
		if os.path.exists(path):
			os.remove(path)


def write(actcd_list):
	paths = ['add.txt', 'current.txt', 'total.txt', 'cure.txt', 'die.txt']
	# 如果存在文件，就删除文件
	is_exist(paths)

	for path, data in zip(paths, actcd_list):
		f = open(path, mode="a")
		for add in data:
			f.write(str(add) + ' ')
		f.close()
	f = open("province.txt", mode="a", encoding="utf-8")
	for province in province_name_list:
		f.write(province + ' ')
	f.close()
	# add_pic(province_name_list,add_list)
	# 保存数据
	actcd_list = [add_list, current_list, total_list, cure_list, die_list]
	write(actcd_list)

	# print(add_list)
	# print(current_list)
	# print(total_list)
	# print(cure_list)
	# print(die_list)


today = datetime.date.today()


# 读取数据
def read(path):
	f = open(path, "r")
	datas = list(map(int, f.readline().strip().split(' ')))
	print(datas, len(datas))

	f = open("province.txt", "r", encoding="utf-8")
	provinces = list(map(str, f.readline().strip().split(' ')))
	print(provinces, len(provinces))
	return datas, provinces
