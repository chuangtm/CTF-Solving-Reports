# CTF-Solving-Reports
　[[Root-Me](https://www.root-me.org/)] [[Cryptanalysis](https://www.root-me.org/en/Challenges/Cryptanalysis/)] [[Pixel Madness](https://www.root-me.org/en/Challenges/Cryptanalysis/Pixel-Madness-86)] [[解题报告](http://exp-blog.com/2019/01/02/pid-2699/)]

------

从题目就可以猜测，这题与像素图有关。

但是题目提供的数据比较奇怪，我最开始以为是用十六进制数表示的像素点，但是 `1x1` 这种数据明显不是十六进制，而且如果是十六进制，每行的像素个数也不同，不可能构成图片。

题目 Clue 提示了 `0 = #FFFFFF（白色）` ， `1 = #000000（黑色）` ，亦即说明图片只有黑白两种颜色，很可能所给的数据应该只有 0 和 1 两种有效像素。

结合考虑，判断 `0xa` 应该是表示连续 a 个 0，而 `1xb` 则表示 连续 b 个 1 。

以这种方式数了一下每行的 01 总个数，都是刚好 100 个，就可以确定判断没错了，这种编码方式实际上就是 `游程长度编码`（注意第 8 行其实是 101 个，多了一个像素，我删掉了末尾的 0 使其对齐，不然无法绘图）。

知道编码方式，那么只需要对其解码，然后绘制成图片就可以了。绘制图片后密码就在里面，图片很多噪点，但还是能勉强看到密码是 `SOLUTION` （注意密码要全大写字母）。

我简单写了个 python 程序对题目给出的图片数据进行解码，并将其还原成图片，贴在下面：

```python
# 被压缩的图片像素(RLC:游程长度编码)
# 如：
#	0x3 表示连续 3 个 0
#	1x2 表示连续 2 个 1
rlc_pixels = [
	'0x3+1x1+0x1+0x1+0x7+1x2+0x15+1x1+0x8+1x1+0x8+1x1+0x1+1x1+0x1+1x1+0x1+1x1+0x1+1x1+0x3+1x1+0x1+1x1+0x3+1x1+0x1+1x4+0x2+1x1+0x25', 
	'0x2+1x1+0x4+1x1+0x4+1x3+0x1+1x2+0x2+1x8+0x11+1x4+0x1+1x3+0x6+1x2+0x4+1x1+0x4+1x2+0x7+1x4+0x4+1x2+0x7+1x2+0x3+1x2+0x3', 
	'0x3+1x1+0x2+1x1+0x2+1x1+0x11+1x2+0x2+1x3+0x7+1x1+0x4+1x2+0x2+1x2+0x7+1x1+0x6+1x1+0x2+1x1+0x4+1x3+0x1+1x1+0x4+1x1+0x2+1x1+0x2+1x1+0x3+1x1+0x2+1x3+0x2+1x2+0x3', 
	'1x1+0x2+1x1+0x4+1x1+0x2+1x1+0x1+1x1+0x2+1x1+0x2+1x1+0x1+1x2+0x2+1x2+0x1+1x2+0x3+1x1+0x3+1x1+0x2+1x2+0x1+1x3+0x3+1x1+0x2+1x1+0x4+1x2+0x1+1x1+0x4+1x1+0x3+1x2+0x12+1x2+0x1+1x1+0x3+1x7+0x3', 
	'0x3+1x1+0x7+1x1+0x1+1x1+0x4+1x1+0x2+1x2+0x2+1x2+0x4+1x1+0x2+1x1+0x1+1x2+0x1+1x8+0x1+1x1+0x4+1x1+0x5+1x1+0x3+1x2+0x2+1x1+0x1+1x2+0x2+1x1+0x3+1x2+0x9+1x1+0x1+1x2+0x2+1x3+0x2+1x1', 
	'0x7+1x1+0x4+1x1+0x4+1x1+0x1+1x1+0x1+1x7+0x3+1x1+0x1+1x2+0x3+1x1+0x1+1x6+0x1+1x1+0x3+1x1+0x2+1x1+0x14+1x2+0x8+1x1+0x10+1x2+0x3+1x2+0x1+1x1+0x1', 
	'0x6+1x5+0x4+1x1+0x7+1x1+0x2+1x1+0x3+1x2+0x4+1x1+0x8+1x1+0x3+1x2+0x1+1x2+0x3+1x1+0x8+1x1+0x2+1x2+0x1+1x1+0x3+1x7+0x5+1x2+0x2+1x1+0x2+1x2+0x3', 
	'0x1+1x1+0x2+1x1+0x1+1x2+0x5+1x1+0x6+1x2+0x3+1x1+0x2+1x1+0x1+1x2+0x20+1x8+0x1+1x1+0x1+1x1+0x4+1x2+0x3+1x1+0x2+1x2+0x3+1x2+0x7+1x2+0x3+1x2+0x3', # 原数据的这行多了一个像素，我把最后的 0x4 改成了 0x3
	'0x2+1x1+0x3+1x5+0x5+1x2+0x7+1x1+0x4+1x2+0x2+1x1+0x2+1x2+0x1+1x1+0x3+1x1+0x6+1x2+0x2+1x2+0x3+1x2+0x2+1x3+0x1+1x1+0x6+1x3+0x3+1x5+0x3+1x1+0x4+1x1+0x5', 
	'0x4+1x2+0x3+1x2+0x3+1x1+0x5+1x2+0x2+1x1+0x1+1x1+0x1+1x1+0x1+1x2+0x9+1x1+0x3+1x1+0x2+1x1+0x1+1x1+0x2+1x1+0x1+1x2+0x2+1x1+0x2+1x1+0x1+1x1+0x4+1x3+0x1+1x1+0x2+1x2+0x3+1x2+0x3+1x1+0x5+1x1+0x4+1x1+0x2', 
	'0x6+1x5+0x4+1x1+0x1+1x1+0x2+1x2+0x6+1x1+0x1+1x7+0x4+1x3+0x3+1x1+0x4+1x1+0x2+1x2+0x4+1x1+0x6+1x1+0x6+1x8+0x3+1x1+0x5+1x1+0x7', 
	'0x2+1x1+0x3+1x6+0x4+1x1+0x1+1x3+0x4+1x1+0x2+1x2+0x4+1x1+0x5+1x1+0x2+1x1+0x3+1x2+0x3+1x1+0x2+1x3+0x1+1x1+0x2+1x2+0x3+1x3+0x2+1x3+0x9+1x1+0x4+1x2+0x7+1x2'
]

IMAGE_WIDTH = 100	# 图片宽度（可以从压缩的像素数据数出来，每行的 0、1 个数之和均为 100）
IMAGE_HEIGHT = 12	# 图片高度（可以从压缩的像素数据数出来，共12行）
WHITE_RGB = (255, 255, 255)		#FFFFFF = 0
BLACK_RGB = (0, 0, 0)			#000000 = 1

# 对图片解码，还原成 100 x 12 的 RGB 像素图
pixels = []
for row in rlc_pixels :
	cols = row.split('+')

	for col in cols :
		bw = col.split('x')
		color = bw[0]
		num = int(bw[1])

		for n in range(num) : 
			rgb = WHITE_RGB if bw[0] == '0' else BLACK_RGB
			pixels.append(rgb)


# 绘制图片
from PIL import Image
img = Image.new("RGB", (IMAGE_WIDTH, IMAGE_HEIGHT))
img.putdata(pixels)
img.save('SOLUTION.png')
img.show()
```

![](https://github.com/lyy289065406/CTF-Solving-Reports/blob/master/rootme/Cryptanalysis/%5B06%5D%20%5B15P%5D%20Pixel%20Madness/imgs/01.png)

------

## 版权声明

　[![Copyright (C) 2016-2019 By EXP](https://img.shields.io/badge/Copyright%20(C)-2006~2019%20By%20EXP-blue.svg)](http://exp-blog.com)　[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
  

- Site: [http://exp-blog.com](http://exp-blog.com) 
- Mail: <a href="mailto:289065406@qq.com?subject=[EXP's Github]%20Your%20Question%20（请写下您的疑问）&amp;body=What%20can%20I%20help%20you?%20（需要我提供什么帮助吗？）">289065406@qq.com</a>


------