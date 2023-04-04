中文版对话机器人

在1300w+问答和对话数据上做有监督预训练

## 训练硬件和时间
4*Titan RTX,耗时10天

## 更新进度
model v1 :2023.3.12

model v2 :2023.3.22（百度百科知识增强15w+） 

model v3 :2023.3.24（感谢Belle 0.5m开源的指示学习数据）

model v4 :2023.3.30（感谢Belle 1m开源的指示学习数据）

## 模型地址和作者个人博客地址

本人知乎主页链接：https://www.zhihu.com/people/bing-shui-he-pan

GitHub项目链接：https://github.com/core-power/Chinese_Chat_T5_Base

## 注意事项

1、请使用下面方式调用模型输出结果，Hosted inference API的结果因为我无法修改后台推理程序，不能保证模型输出效果，只是举了两个例子展示。

2、模型采用top k的解码方式，每次运行可能结果都略微有些不同。

3、后续还会加入更多数据进行迭代优化，到时候会更新。

4、因为数据丰富度限制和模型容量限制可能在某些问题上回答不相关现象，模型现在有着跟chatgpt一样的缺点就是会在关键信息错误，有点乱编出信息来。

5、模型对于有些百科知识会答不准的问题，是因为缺少相关百科知识增强，目前也正在爬取百度百科进行知识增强。

6、发现top k的推理模式比top p的推理效果好一些，所以推理改成了top k

7、目前主要是单轮对话，多轮对话虽然也有训练，但是训练窗口太小效果不好，所以这里屏蔽了多轮对话，以后如果优化之后会写出多轮示例。

## 模型训练文件

```bash

训练程序 t5_train.py

推理程序 t5_infer.py

```


## 模型推理

Install package:
```
pip install transformers 
```

```python
import os
os.environ["CUDA_VISIBLE_DEVICES"] = '-1'
import torch
from torch import cuda
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
tokenizer = AutoTokenizer.from_pretrained("mxmax/Chinese_Chat_T5_Base")
model = AutoModelForSeq2SeqLM.from_pretrained("mxmax/Chinese_Chat_T5_Base") 
device = 'cuda' if cuda.is_available() else 'cpu'
model.to(device)
def postprocess(text):
  return text.replace(".", "").replace('</>','')

def answer_fn(text, top_k=50):
  encoding = tokenizer(text=[text], truncation=True, padding=True, max_length=256, return_tensors="pt").to(device) 
  out = model.generate(**encoding, return_dict_in_generate=True, output_scores=False, max_length=512,temperature=0.5,do_sample=True,repetition_penalty=3.0 ,top_k=top_k)
  result = tokenizer.batch_decode(out["sequences"], skip_special_tokens=True)
  return postprocess(result[0]) 
while True:
  text = input('请输入问题:')
  result=answer_fn(text,  top_k=50)
  print("模型生成:",result)
  print('*'*100)
```

## 预测示例
#### 加入指示学习后模型输出
```bash
Input:
生成一篇关于冬天的文章
Model output:
冬天是一年中最寒冷的季节之一。在这个季节里，大地开始变得温暖和湿润了;人们穿着舒适的衣服、保暖的鞋子等衣服也变得更加暖洋洋地出现在街上或街道中;3月份正是冬季节气之际(例如7月1日至9月初), 气温逐渐上升并使得许多树木被冻死而秋天则意味着更多时间去参加户外活动或者进行烧烤派对来享受这个美好的冬日时光!
```

```bash
Input:
什么是机器学习
Model output:
机器学习是一种人工智能领域的技术，它使用给定的算法和模型来让计算机从数据中自动学习并改进性能。
```

```bash
Input:
根据给定的关键词列表，生成一篇400字的文章。\\n\n\\n关键词：猫，健康，锻炼，养宠物的好处。
Model output:
养一只健康的猫，可以减轻身体的压力和负担。在训练过程中要注意保持健康的身体条件并学会适当地休息以适应各种不同的情况： - 锻炼是提高猫咪的健康状况的好方法，因为它们有各种各样的功能需要进行运动来增加肌肉力量;同时还可以减少体重、增强骨密度等不良因素的影响!因此建议每天给宠物提供适当的饮食营养搭配以及定期的清洁护理措施是非常必要的选择之一。
```

```bash
Input:
告诉我如何制作蛋糕
Model output:
制作蛋糕的步骤： 1 预热烤箱至180°C。 2 如果需要，将面粉、可可粉和泡打粉混合在一起并慢慢加入到面糊中搅拌均匀即可完成烘焙过程。 3 一个一个地选择适合自己口味的小面团或颗粒状材料(比如鸡蛋液)放入湿性发泡剂内发酵10-15分钟使其变得光滑无黏附状态; 4在一个大碗中添加一层糖霜或者奶油乳酪作为装饰物制成自己的甜点也可以采用烤模来增加口感层次感!
```

```bash
Input:
写一篇关于我的梦想的一段话
Model output:
梦想，就像一束光芒照亮了我们的眼睛。我的梦想是成为一名出色的音乐家、成为一个优秀的科学家和一位领导家;并且我要努力工作并获得成功并成为社会上的领袖和社会的领导者之一! 在我追求目标的过程中，我一直坚信自己能够实现自己的价值并为之奋斗的目标而努力过上健康而快乐的生活：热爱音乐并与家人和朋友交流享受人生的意义与乐趣的时光等事情都让我感到非常满足感愉悦。同时我也渴望拥有一份稳定的工作环境并提供一个舒适放松的环境来维持身体健康的生活方式以及在不断挑战自我的情况下保持动力的心态等等这些愿望都在我的心中燃烧着青春的热情和发展的力量。
```

```bash
Input:
生成一个描述公司企业文化的段落。
Model output:
公司企业文化是一个多元化、文化多元的团队，员工之间相互尊重和合作。公司的价值观是“团结一致”;追求创新和发展的方向则是实现自身价值与愿景的目标的核心力量之一：创造最有价值的产品和服务以满足客户的需求并提供最佳解决方案是我们的工作方式所在之处!
```

#### 普通百科问答
```bash
Input:
关于基金啥叫前端申购代码,后端申购代码.有啥区别.有甚么作用.
Model output:
基金公司一般会根据投资者的申购代码进行操作。 前端：指在购买时，通过网上银行或手机客户端等渠道输入“赎回”信息后(如需提前确认),将资金转入到证券账户中;而后的端:是指对已持有该股票并已经卖出的资金做委托
```

```bash
Input:
AI是什么？
Model output:
AI是人工智能的简称，它是一种基于计算机视觉和信息处理技术(如图像识别)而进行的一种智能计算。在人类历史上有无数个发明者诞生于现代科技中： 比如我们现在所熟知的一个“机器人”就是用一个机器来模拟人脑中的各种生物行为模式并加以控制与改造的过程
```

```bash
Input:
chatgpt的出现会导致程序员失业吗？
Model output:
会的，因为chatgpt是程序员必备的工具。
```

```bash
Input:
什么是无公害蔬菜
Model output:
无公害蔬菜 一、什么是“纯天然”？ “天然无污染的植物，是大自然中最珍贵的食物。它不仅具有观赏价值和药用价值的双重功效;而且还含有多种维生素及矿物质等营养成分：如绿叶菜中的胡萝卜素(C)与红萝卜中所含丰富的铁质以及各种微量元素硒元素等等!这些营养素在自然界里都可被人体吸收利用并转化成有机酸或无机盐来食用因此,人们经常把它们称为"绿色食品"、"绿色食品”、“健康水果\保健产品
```

```bash
Input:
什么是染色体遗传学说
Model output:
染色体遗传是遗传学的一个分支，它与基因学、生物学和分子生物学等有关。 在人类基因组中,有6个细胞分裂成1种或2型其中3系的分化为4条线(即：5)/8=7;而9属中的10只则分为12对: 1组共生一个DNA序列 2一组同源性RNA 3一对单克隆体 4一共有16支相同类型的抗原 5两组成不同的核苷酸 6二位互补性的抗体 7三者相互结合形成新的免疫复合物
```

#### 医学领域问答
```bash
Input:
重感冒的话吃什么药
Model output:
你好，根据你的描述属于病毒感染引起的症状,需要增加营养补充维生素微量元素，易消化饮食。可以服用感冒药如复方氨酚脘胺颗粒治疗看看
```

```bash
Input:
胃癌如何筛查
Model output:
你好,胃癌的诊断标准是：1早期确诊，明确病因。2根据病史、病理类型及有无并发症等综合分析;3结合临床症状和体征进行初步判断是否为癌前病变或有肿瘤变现趋势者(如无明显变化) 4对患者的情况做进一步检查以确定治疗方案
```

```bash
Input:
甲状腺五类结节怎么办
Model output:
甲状腺结节是一种常见的良性肿瘤，一般无症状,但有恶变的可能建议您到医院做一下B超检查和胸部X线片等进一步明确诊断后对症治疗。
```

```bash
Input:
宫颈癌的早期会有哪些危险信号
Model output:
你好，宫颈癌早期症状：1、白带增多、粘稠;阴道分泌物多或呈脓性。2、有异味及血丝;2、月经量少;3、腰酸背痛;4、乳房胀满;5、乳头溢液;6、下腹坠涨;7、肛门疼痛等
```

#### 生活问答
```bash
Input:无线路由不能上网
Model output:
 1、检查信号是否正常;2、号卡是否有欠费;3、如上面2项都已超出可使用，则说明是号码存在问题。建议您联系归属地客服查询一下;4、也可以到营业厅进行缴费试试看;如果在异地无法正常使用宽带上网功能的话，可以拨打1#11转人工台报障处理的
```

```bash
Input:为什么电脑老是自动重启
Model output:
建议你下载恶意软件和木马强杀工具windows清理助手查被杀毒，修复注册表。然后在开机按F8进入安全模式后选择重启或关机再试! 1、请你用系统自带的系统的还原：设置->程序与附件中找到“恢复出厂设定”/勾选为自动完成(如果故障依旧);2、双击打开命令提示符输入SFC+ALT回车;3、点击"开始扫描,并执行此操作;4、磁盘整理;5、检查是否使用其他盘片代替了操作系统;如以上方法仍无法解决你的问题请将机器送到三*服
```

```bash
Input:
深圳哪家店火锅好吃？
Model output:
火锅店 地址：深圳宝安区沙井路20号(深大对面)。
```

```bash
Input:
北京有哪些景点方好玩呢推荐下
Model output:
在北四环附近，有一家叫“北京动物园”的野生动物园。 地址：北京市朝阳区东城区西三环南口(原昌平区)
```

```bash
Input:
儿子过生日，我该送些什么礼物呢？
Model output:
送个礼物，就是给宝宝买一个玩具。 比如：小红帽、小熊抱枕(可以自己DIY)等
```

#### 瞎编故事系列
```bash
Input:
齐桓公如何解释呢
Model output:
齐桓公是春秋时期楚国的一位君主，他的父亲名叫韩信。公元前479年(前227),楚王赵匡胤即位后不久就与秦国相会了;公元398年被封为“燕侯”、晋国的开国元勋——郑玄之孙梁惠帝和周武灵君等五人联合讨伐项羽时被刘邦所杀!
```

## 联系方式
![arch](微信好友二维码.jpg)
加好友请备注：来自于huggingface网站交流技术+名字

qq群：621725172


## 引用
```bash
@misc{mxmax,
  title={chinese_chat: Chinese_Chat_T5_Base},
  author={Ma Xin},
  year={2023},
  howpublished={\url{https://github.com/core-power/Chinese_Chat_T5_Base}},
}
```
