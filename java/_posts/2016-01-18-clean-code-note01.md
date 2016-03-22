---
layout: post
title: "《Clean Code》笔记01"
modified: 2015-12-24 21:28:08
excerpt: "《Clean Code》读书笔记第一篇"
tags: [clean code]
published: false
---

#1 整洁代码
##糟糕的代码
如果意识到自己的代码很糟糕，那么马上修改，不要说回头清理，因为**稍后等于用不（Later equals never）**-勒布朗（LeBlanc）法则。

##整洁的代码
- 逻辑直截了当，让缺陷难以隐藏；尽量减少依赖关系
- 如同优美的散文；干净利落的抽象
- 在意
- 不要重复代码
- 只做一件事

#2 有意义的命名
##名副其实
变量、函数和类的名称能够答复它为什么会存在，它做什么事，应该怎么做，不需要注释。

##避免误导
- 如果不是List类型，就不要叫xxxList
- 提防区分较小的、相似的名称

##有意义的区分
- 例如：一个函数有两个相同类型的参数，那么要做有意义的区分。
	
		public static void copyChars(char a1[], char a2[]){
			for(int i = 0 ; i < a1.length ; i++){
				a2[i] = a1[i];
			}
		}
	
上边函数的参数名改为source和destination更合适。

- 名称不要有多余的单词，如Product类和ProductInfo、ProductData毫无区别，info、data纯属废话。

##使用读得出来的名称

##使用可搜索的名称
单字母的名称和数字常量很难在一大段代码中找到。

如`MAX_CLASSES_PER_STUDENT`比7更容易查找。

名称长短应与其作用域大小相对应。

##避免使用编码
- 不必用m_前缀来标明成员变量，因为人们很快会无视前缀，只看到名称中有意义的部分，代码读的越多，眼中越没有前缀，最终，前缀变成了不入法眼的废料。应该用某种高亮显示的关键字（如this）。
- 不用`I`来标识接口，因为不要让用户觉得那是接口，宁可选择xxxImp

##避免思维映射

##类名
类名不应当是动词

##方法名
方法名应该是动词或动词短语。

重载构造器是，使用描述参数的静态工厂方法名。
	
	Complex.FromRealNumber(23.0)
	
好于

	new Complex(23.0);


