---
title: 关于canvas的图片获取及python处理
date: 2022-06-03 11:51:13
tags:
- python
categories:
- python常用函数
---
### 获取canvas图片的对应base64的uri（echart图、v-chart图
canvas元素.toDataURL()获取对应canvas的base64 uri的链接

### 前端处理生成的uri，可以生成一张图片
```javascript
<img src="base64uri"/>
```
### python处理canvas生成的base64 uri
```python
def generate_img(self, img_path, img_uri):
	"""
	根据base64生成图片
	:param: img_path:生成的图片路径
	:param: img_uri:图片的base64 uri
	"""
	try:
		#截取uri的data:image/png;base64后端的uri
		img_uri = img_uri.split(",")[1]
	    imgdata = base64.b64decode(img_uri)
	    with open(img_path, mode="wb") as f:
	        f.write(imgdata)
	except Exception as e:
	    device_monitor_output_log.error("generate_img error:{}".format(e))
```
