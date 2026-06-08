---
title: 游戏设计模式——单例模式
date: 2026.6.5
updated:
tags: 你也想获得技术的力量吗 
categories: 游戏设计模式
keywords:
description: 单例模式相关知识随记
top_img: /img/preview.jpg
comments:
cover:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink: false
aside: true
abcjs:
noticeOutdate:
---

  
---

# 引言

	   设计模式是程序艺术家们组织数据，类，对象的方法和模版，更是一种思想。在适当情景采取适当的设计模式，往往能在完美高效地解决需求的同时，让代码变得优雅而简洁。


# 单例模式

## 一：什么是单例模式？

  ### 想像一个情景：

   你需要给游戏添加音效，让不同的游戏角色拥有自己的声音，需要控制每个人用什么声音，需在适时改变声音大小。

   那我们应该给每个角色都写一个切换音效，控制音量的方法吗？那太繁琐了，也不优雅
   要是能用一个管理员统一管理这些音频就好了

  ### 单例模式正是来解决这种重复性工作的问题的，通过一个全局唯一的实例，把所有与之相关的对象统一管理，并向外部开放管理方法，外部只需引用管理方法便能轻松管理这些对象，所以负责管理某个方面的单例又被称为“XXManager”
  
  ### 事实上单例有诸多优势：
   -  **避免重复创建**：管理器类只需要一份，不需要每个场景都重新创建
   -  **全局访问**：任何脚本都可以通过 `ClassName.Instance` 快速获取引用
   -  **跨场景持久化**：配合 `DontDestroyOnLoad`，数据可以在场景切换时保留
   -  **减少性能开销**：避免大量使用 `FindObjectOfType` 查找对象

  ### 正因如此，使用单例模式时应保持其全局性和唯一性，不然会产生数据安全问题

## 二：单例应包含什么？
  
 - **一个公共静态成员属性或方法**  保存并向外部暴露其唯一的实例（Instance）
 - **一个获得实例的方法** 在适时将实例指向单例本身，并应确保其唯一
 - **若干个业务方法** 供外部通过方法实现业务逻辑

   ### 应注意的是，一个优秀的单例设计应随安全性要求不同或承载业务逻辑不同而灵活变化，以达到需求和性能开销的平衡


## 三：根据需求兼安全的分类

- ### 基础单例：

  ##### 实现单例的最基础形式，适合纯数据管理

	{% code %} 
		public class TestManager
	{
	    private static TestManager instance;
	
	    public static TestManager Instance //属性和方法任选其一
	    {
	        get
	        {
	            if (instance == null)
	            {
	                instance = new TestManager();
	            }
	
	            return instance;
	        }
	
	    }
	
	    public static TestManager GetInstance() //属性和方法任选其一
	    {
	        if (instance == null)
	        {
	            instance = new TestManager();
	        }
	
	        return instance;
	    }
	}
	   
	{% endcode %}


   #### **优点**：
   - **代码简洁，易于理解**
  

   #### **缺点**：
   - **不继承Mono，不支持生命周期，协程等功能**
   - **非线程安全，多线程访问可能导致数据不一致**

- ### 单例泛型单例基类（Manager Of Manager）：

  #### 为避免重复写单例固定格式内容，构建带泛型参数T的Manager基类，只需继承基类并将子类作为参数传入即可获得单例的功能

	{% code %} 
		
		public class BaseManager<T>  where T : class ,new()
		{
			private static T instance;
		
			public static T Instance
			{
				get
				{
					if (instance == null)
					{
						instance = new T();
					}
		
					return instance;
				}
		
			}
		
			public static T GetInstance()
			{
				if (instance == null)
				{
					instance = new T();
				}
		
				return instance;
			}
		}
		
	{% endcode %}


  #### **优点：**
   - **避免重复代码，便于创建大量单例**

  #### **缺点：**
   - **不继承Mono，不支持生命周期，协程等功能**
   - **非线程安全，多线程访问可能导致数据不一致**

- ### 继承Mono的单例泛型单例基类：

  #### 在基类的优点上拓展Mono的功能，适合管理游戏实例，但要记得将脚本挂好


	{% code %} 
		public class BaseManager_Mono<T> :MonoBehaviour where T: MonoBehaviour
	{
	    private static T instance;
	
	    public static T Instance
	    {
	        get
	        {
	            return instance;
	        }
	
	    }
	
	    public static T GetInstance()
	    {
	        return instance;
	    }
	
	    protected virtual  void Awake()
	    {
	        instance = this as T;
	    }
	
	
	}

  {% endcode %}

  #### **优点：**
  - **避免重复代码，便于创建大量单例**
  - **拥有Mono的功能**
  - **通过Awake等生命周期方法实现周期内自动维护单例**

  #### **缺点：**
  - **必须被手动挂载才能生效**
  - **Mono带来的可挂载性导致一个实例可能重复挂载多个单例脚本，破坏唯一性**
  - **无法跨场景保持，但可以在子类中添加**
  - **非线程安全，多线程访问可能导致数据不一致**
  

  #### 在此基础上添加自动创建实例并添加脚本的功能：

	{% code %} 
		public class BaseManager_Mono<T> :MonoBehaviour where T: MonoBehaviour
	{
	    private static T instance;
	
	    public static T Instance
	    {
	        get
	        {
	            if (instance == null)
	            { 
	                GameObject obj = new GameObject(typeof(T).Name);
	                instance = obj.AddComponent<T>();
	
	                DontDestroyOnLoad(obj);
	            }
	            return instance;
	        }
	
	    }
	
	    public static T GetInstance()
	    {
	        return instance;
	    }
	
	    protected virtual  void Awake()
	    {
	        instance = this as T;
	    }
	
	
	}

  {% endcode %}

  #### **优点：**
  - **避免重复代码，便于创建大量单例**
  - **拥有Mono的功能**
  - **跨场景持久化，DontDestroyOnLoad保证实例不被刷掉**
  - **通过Awake等生命周期方法实现周期内自动维护单例**
  - **将关键生命周期方法修饰为保护（protected）虚（virtual）方法，避免子类丢失维护功能**

  #### **缺点：**
  - **Mono带来的可挂载性导致一个实例可能重复挂载多个单例脚本，破坏唯一性**
  - **场景来回切换时会自动添加单例实例，破坏唯一性**
  - **非线程安全，多线程访问可能导致数据不一致**




 ## 四：补充

 ### 对于不继承Mono的单例，会存在构造函数带来的唯一性问题：

  #### 单例类是可以被new出来的，这样产生的新实例必定破坏唯一性
  
  #### 但作为一个程序员，单例不应该被new应当是共识，所以不再长篇解释如何解决，只提供思路
  
  #### 对于基础单例（不通过继承基类获得单例效果），直接将构造函数设为私有即可，然对于单例基类，直接私有构造会导致基类无法自动创建子类对象给instance，所以可以通过基类反射子类获得唯一的无参构造创建对象

