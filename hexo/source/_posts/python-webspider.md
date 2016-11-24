---
title: Python网页爬虫
date: 2016-01-27 17:43:20
tags:
	- Python
---

> 最近在自学Python，感谢[廖雪峰前辈的Python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)。看了这教程，算是对Python语法有了一点基础，越发地迷上Python，想继续深入学习Python，在网上找了下资料，发现鱼C网站有Python视频教程，更难能可贵的是居然有百度网盘的分享地址，鱼C的教程我并不陌生，之前就看过鱼C出品的汇编语言教程，觉得挺不错的。所以就想着保存一份到自己的网盘上，但是10页数据一共97个视频，难道你还要一个一个点击进入，再点击查看百度网盘的下载地址吗？想想我都觉得要疯了，所以用Python写了这个小脚本，负责抓取所有的百度网盘链接和提取密码，然后调用Mac平台的open命令打开Safari，剩下的事情就简单多了，直接copy提取密码，Command+V然后回车，进去后再保存到自己的网盘。

``` Python
#!/usr/bin/python
#coding:UTF-8
 
import urllib
import re
import os
import sys
 
class WebSpider(object):
	def __init__(self):
		pass
 
	def startRun(self, baseUrl, startPage, endPage):
		for page in range(startPage, endPage+1):
			url = baseUrl + str(page)
			links = self.getPageLinksWithUrl(url)
			print len(links)
			for link in links:
				bdInfo = self.getBaiduYunInfo(link)
				if bdInfo == None:
					print 'The url is parse error: %s' % link
					continue
				print bdInfo
				bdUrl = bdInfo[0]
				self.openSafari(bdUrl)
 
	def getPageLinksWithUrl(self, url):
		html = self.getHtmlWithUrl(url)
		links = self.getLinksWithHtml(html)
		return links
 
	def getHtmlWithUrl(self, url):
		fp = urllib.urlopen(url)
		html = fp.read()
		return html
 
	def getLinksWithHtml(self, html):
		links = []
		pattern = re.compile('<div\\s*class=[\'\"](article|tagleft)[\'\"]>\\s*<h2>\\s*<a\\s*href=[\'"]([^"\']*?)[\'"]\\s+rel="bookmark"')
		items = re.findall(pattern, html)
		for item in items:
			links.append(item[1])
		# remove repeat
		links = list(set(links))
		return links
 
	def saveLinksToFile(self, file):
		pass
 
	def getBaiduYunInfo(self, url):
		html = self.getHtmlWithUrl(url)
		pattern = re.compile('<p\\s+class=[\'\"]part[\'\"]\\s+id=[\'\"]download_button_part[\'\"]>(.*?)</p>')
		items = re.findall(pattern, html)
		if len(items) == 0:
			# fenye
			paging = re.compile('<div\\s+class=[\'\"]fenye[\'\"]>(.*?)</div>')
			fenye = re.findall(paging, html)
			if len(fenye) == 0:
				return None
			pagecount = len(fenye[0].split('</a>')) - 1
			if pagecount <= 0:
				return None
			pageUrl = r'%s/%d' % (url, pagecount)
			return self.getBaiduYunInfo(pageUrl)
		_, bdInfo = items[0].split('|')
		return self.parseBaiduYunATag(bdInfo)
		
	def parseBaiduYunATag(self, string):
		pattern = re.compile('(href="([^\'\"]*)"|：(\\w{4}))')
		items = re.findall(pattern, string)
		bdUrl = items[0][1]
		bdPassword = items[1][2]
		return (bdUrl, bdPassword)
 
	def openSafari(self, url):
		if sys.platform != 'darwin':
			return
		cmd = r'open %s' % url
		os.system(cmd)
 
 
def main():
	url = r"http://blog.fishc.com/category/python/page/"
	spider = WebSpider()
	spider.startRun(url, 1, 1)
 
if __name__ == '__main__':
	main()
```