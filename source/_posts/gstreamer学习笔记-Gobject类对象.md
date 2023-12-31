---
title: GStreamer学习笔记--GObject类对象
date: 2021-07-17 16:48:47
tags: [GStreamer,GObject]
categories: Internship
description: GObject 设计用于直接在C程序中使用以提供面向对象的基于C的API，并通过与其他语言的绑定来提供透明的跨语言互操作性，例如PyGObject。
---

[TOC]

# GStreamer学习笔记--GObject类对象

## Gobject类定义

维基百科：**GObject**，是一个在[LGPL](https://zh.wikipedia.org/wiki/LGPL)下发布的[自由](https://zh.wikipedia.org/wiki/自由软件)[软件库](https://zh.wikipedia.org/wiki/软件库)，它提供了一个轻便的[对象系统](https://zh.wikipedia.org/w/index.php?title=对象系统&action=edit&redlink=1)并支持透明的多语言互通。GObject被设计为可以直接使用在[C](https://zh.wikipedia.org/wiki/C语言)程序中，也可以被封装至其他语言，例如[C++](https://zh.wikipedia.org/wiki/C%2B%2B)，[Java](https://zh.wikipedia.org/wiki/Java)，[Python](https://zh.wikipedia.org/wiki/Python)，以及可以生成C代码的[Vala](https://zh.wikipedia.org/wiki/Vala)（由此大大简化了`GObject`代码的书写）等等。

`Gstreamer`框架是基于插件的，同时插件是可以动态的注册、创建，`gstreamer`基于`Gobject`开发，下面来了解一下`gstreamer`是如何通过`Gobject`完成自定义类的注册。

在每个类的`c`文件中，都会有以下这样的一个宏定义：

```c
G_DEFINE_TYPE (GstV4l2Allocator, gst_v4l2_allocator, GST_TYPE_ALLOCATOR);
```

`G_DEFINE_TYPE`是一个宏定义，那么这个`G_DEFINE_TYPE`宏是如何完成向`Gobject`系统完成类的注册呢？

将`G_DEFINE_TYPE`展开可以看到以下代码：

```c
#define G_DEFINE_TYPE(TN, t_n, T_P)
/******* 
 *TN  ---> TypeName
 *t_n ---> type_name
 *T_P ---> TYPE_PARENT
 *_f_ ---> 0
 *_c_ ---> {}
******/
/*****   以下为宏展开   *****/
static void     type_name##_init              (TypeName        *self); 
static void     type_name##_class_init        (TypeName##Class *klass); 
static gpointer type_name##_parent_class = NULL;
static gint     TypeName##_private_offset;
 
static void     type_name##_class_intern_init (gpointer klass)
{
  type_name##_parent_class = g_type_class_peek_parent (klass);
  if (TypeName##_private_offset != 0)
    g_type_class_adjust_private_offset (klass, &TypeName##_private_offset);
  type_name##_class_init ((TypeName##Class*) klass);
}
 
static inline gpointer
type_name##_get_instance_private (TypeName *self)
{
  return (G_STRUCT_MEMBER_P (self, TypeName##_private_offset));
} 
 
GType 
type_name##_get_type (void) 
{ 
  static volatile gsize g_define_type_id__volatile = 0;
  /* Prelude goes here */
  if (g_once_init_enter (&g_define_type_id__volatile))
    {
      GType g_define_type_id =
        g_type_register_static_simple (TYPE_PARENT,
                                       g_intern_static_string (#TypeName),
                                       sizeof (TypeName##Class),
                                       (GClassInitFunc)(void (*)(void)) type_name##_class_intern_init,
                                       sizeof (TypeName),
                                       (GInstanceInitFunc)(void (*)(void)) type_name##_init,
                                       (GTypeFlags) flags);
      { /* custom code follows */
      		{_C_;}
        /* following custom code */
      }
      g_once_init_leave (&g_define_type_id__volatile, g_define_type_id);
    }
  return g_define_type_id__volatile;
} /* closes type_name##_get_type() */
```

## 向Gobject系统注册类

`G_DEFINE_TYPE`定义如上，那么，最终它是如何向`Gobject`系统注册该类的呢？
`Gobject`系统为什么知道你新添加了一个名叫`TypeName`的类，是因为你通过`g_type_register_static_simple()`函数告诉它，我这里有一个新类，你登记一下`g_type_register_static_simple()`函数声明如下：

```c
GLIB_AVAILABLE_IN_ALL
GType g_type_register_static_simple     (GType                       parent_type,
					 const gchar                *type_name,
					 guint                       class_size,
					 GClassInitFunc              class_init,
					 guint                       instance_size,
					 GInstanceInitFunc           instance_init,
					 GTypeFlags	             flags);
```

函数声明的前面`GLIB_AVAILABLE_IN_ALL`就是一个`extern`关键词，从函数声明我们可以了解到，向`Gobject`系统注册一个类，需要告诉`Gobject`系统，我现在需要注册一个新类，它父类的类型是`parent_type`，大小是`class_size`，类的初始化函数是`class_init`，类的实例大小以及初始化函数，还有这个类有什么`flags`，通过告诉`Gobject`，它就会将新类登记在线。

通过`G_DEFINE_TYPE`宏的展开可以知道，在`type_name##_get_type()`函数中调用到`g_type_register_static_simple()`函数，那么，究竟是什么时候，程序会向`Gobject`系统注册该新类呢？
比如我们是要注册一个名叫`TestObject`的类，那么就是通过`TestObject_get_type()`函数完成`estObject`的注册登记。
在我们需要创建一个`TestObject`的实例时，会通过调用`g_object_new()`函数完成，在调用`g_object_new`函数，需要传进相应的参数，这个时候，我们就将`TestObject_get_type()`函数的返回值传递给它，即演变成以下：

```c
TestObject *testObject = （TestObject *）g_object_new (TestObject_get_type(), NULL);
```

在创建`TestObject`实例对象的时候，将会调用`TestObject_get_type()`函数得到相应的类型，而在`TestObject_get_type()`函数中，将会先通过`g_once_init_enter()`函数检查`TestObject_get_type()`中的静态变量`g_define_type_id_volatile`是否为0，如果是，则通过`g_type_register_static_simple()`函数向`Gobject`系统登记`TestObject`类，同时返回`object ID`，如果`g_define_type_id_volatile`不为0，则说明已经向`Gobject`系统注册`TestObjec`t类，直接返回`object ID`，这样，即完成了`TestObject`的注册登记。

## 类的构造函数

学习`C++`我们都知道，类是有构造函数的，在创建类实例的时候，会自动调用该类的构造函数，那么，在`Gobject`中，又是怎么调用类的构造函数呢？

以`TestObject`为例，在上面说到通过`g_type_register_static_simple()`函数向`Gobject`系统注册自定义类的时候，就传进了相应的参数，包括类的初始化函数`test_object_class_intern_init()`以及类实例的初始化函数`test_object_init()`，它们两个共同的相当于`TestObject`类的构造函数。从宏定义`G_DEFINE_TYPE`的展开代码中发现以下函数声明以及`test_object_class_intern_init()`函数的定义：

```c

static void     test_object_init              (TestObject      *self); 
static void     test_object_class_init        (TestObjectClass *klass); 
 
static void     test_object_class_intern_init (gpointer klass)
{
  test_object_parent_class = g_type_class_peek_parent (klass);
  if (test_object_private_offset != 0)
    g_type_class_adjust_private_offset (klass, &TestObject_private_offset);
  test_object_class_init ((TestObjectClass*) klass);
}
```

从上述代码我们可以知道，在通过`G_DEFINE_TYPE`向`Gobject`系统注册类，还需要我们实现`test_object_class_init()`和`test_object_init()`函数的定义。`test_object_class_init()`函数是在第一次创建`TestObject`类实例对象的时候调用的，该函数只会调用一次，而`test_object_init()`函数则是每次创建`TestObject`类实例对象都会调用。

 ## 类的继承关系

在`G_DEFINE_TYPE`的展开代码中，可以看到以下代码

```c
static gpointer type_name##_parent_class = NULL;
 
static void     type_name##_class_intern_init (gpointer klass)
{
  type_name##_parent_class = g_type_class_peek_parent (klass);
  if (TypeName##_private_offset != 0)
    g_type_class_adjust_private_offset (klass, &TypeName##_private_offset);
  type_name##_class_init ((TypeName##Class*) klass);
}
```

在这个宏中，可以看到定义了一个静态的全局指针变量`type_name_parent_class`，而`type_name_parent_class`变量是通过`g_type_class_peek_parent()`函数赋值的，`type_name_parent_class`变量代表着什么呢，它就是父类。一般的，会在该源文件新增一个宏，定义如下：

```c
#define type_name##_parent_class parent_class
```

这样就可以通过宏定义`parent_class`直接调用父类函数，而该父类，就是在通过宏定义`G_DEFINE_TYPE`向`Gobject`系统注册类时传进的第三个参数`T_P`。`g_type_class_peek_parent()`函数通过传进的子类指针，查找到注册时候的相应信息，得到父类的类型，而后通过父类类型得到父类信息并返回。

## 类的析构函数

有了相应的构造函数，在构造函数中申请了内存、硬件等资源，自然的，也会类似`C++`的，有相应的析构函数负责资源的释放操作。那么，在`Gobject`系统中，析构函数又是什么回事呢？我们都知道，构造函数是从父类到子类，而析构函数是从子类到父类。在`Gobject`系统中的析构函数又是如何的呢？

之前说到，在通过`G_DEFINE_TYPE`向`Gobject`系统，注册`TestObject`类的时候，需要定义`test_object_class_init()`和`test_object_init()`函数，而在类实例的初始化函数`test_object_init()`中，我们可能申请了一些内存等资源，我们需要在析构函数中释放这些资源，这个时候，需要我们在`TestObject`类初始化函数`test_object_class_init()`覆盖从父类继承的析构函数，具体代码如下：

```

static void
test_object_dispose (GObject * object)
{
	TestObject *testobject = TEST_OBJECT (object);
 
	/*  资源释放*/
 
	/*  调用父类的dispose 函数 */
	G_OBJECT_CLASS (parent_class)->dispose (object);
}
 
static void
test_object_finalize (TestObject * testobject)
{
	g_free(testobject->mem);
	
    /*  调用父类的finalize 函数 */
	G_OBJECT_CLASS (parent_class)->finalize (object);
}
 
static void test_object_init(TestObject * self)
{
	self->mem = g_malloc (1);
}
static void test_object_class_init(TestObjectClass *klass)
{
	GObjectClass *object_class = G_OBJECT_CLASS (klass);
 
	object_class->dispose = test_object_dispose;
	object_class->finalize = test_object_finalize;
 
	...
}
```

由上述代码我们可以知道，在`TestObject`的初始化的时候，将会覆盖从父类继承而来的析构函数，同时在析构函数中释放类实例初始化时占用的资源，同时还有递归调用父类的析构函数。`dispose`函数主要是将在类中占用的资源释放，而`finalize`函数则是有点类似真正的析构函数，将构造函数申请的资源进行释放回收。

既然析构函数也已经有了，析构函数又会是什么时候调用呢？

` JAVA`使用的是垃圾回收的机制，而`Gobjec`t则是使用引用计数的方式。当每个对象创建的时候，将会对其引用计数加一，如果期间被其他对象进行引用，也都会将它的引用计数增加；而当对象被解除引用的时候，引用计数将会减一，当引用计数减为0的时候，将会调用对象的析构函数，进行资源的回收。

`Gobject`的引用计数方式大致如下：

* 使用`g_object_new()`函数进行实例化的时候，对象的引用计数为1；
* 使用`g_object_ref()`函数进行引用对象的时候，对象的引用计数加1；
* 使用`g_object_unref()`函数解除引用的时候，对象的引用计数减1；
* 调用`g_object_unref()`函数进行解引用的时候，如果发现对象的引用计数为0，将会先后调用该对象的`dispose()`函数和`finalize()`函数。

而为什么在`test_object_class_init()`函数中覆盖从父类继承过来的析构函数呢？
因为在`g_object_unref()`函数中调用`dispose()`函数和`finalize()`函数是通过宏定义`G_OBJECT_GET_CLASS取得OBJECT_CLASS`类之后，再调用它的`dispose()`函数和`finalize()`函数，所以需要在`TestObject`的类初始化函数对这两个函数指针进行覆盖，而在`TestObject`类的`dispose()`函数和`finalize()`函数再通过`G_OBJECT_CLASS (parent_class)`取得父类指针，调用父类的析构函数。

## 类的其他设置

在`Gobject`系统中，设置了很多方便的宏，使在使用对象的时候可以更加的方便，在相应的头文件，一般会有如下宏定义：

```c
typedef struct _GstTestObject TestObject;
typedef struct _GstTestObjectClass GstTestObjectClass;
 
/* 获取类型 */
#define GST_TYPE_TEST_OBJECT      (test_object_get_type())
 
/* 类实例类型判断 */
#define GST_IS_TEST_OBJECT(obj)   (G_TYPE_CHECK_INSTANCE_TYPE ((obj), GST_TYPE_TEST_OBJECT))
 
/* 类结构判定 */
#define GST_IS_TEST_OBJECT_CLASS(klass)      (G_TYPE_CHECK_CLASS_TYPE ((klass), GST_TYPE_TEST_OBJECT))
 
/* 获取obj的类型，同时将其转换为GST_TYPE_TEST_OBJECT，并返回指向GstTestObjectClass的指针 */
#define GST_TEST_OBJECT_GET_CLASS(obj)       (G_TYPE_INSTANCE_GET_CLASS ((obj), GST_TYPE_TEST_OBJECT, GstTestObjectClass))
 
/* 检查obj是否是GST_TYPE_TEST_OBJECT类型，如果是，则将返回指向obj成员变量TestObject的指针 */
#define GST_TEST_OBJECT(obj)      (G_TYPE_CHECK_INSTANCE_CAST ((obj), GST_TYPE_TEST_OBJECT, TestObject))
 
/* 检查klass是不是GST_TYPE_TEST_OBJECT类型，如果是，则将返回指向klass成员变量GstTestObjectClass的指针 */
#define GST_TEST_OBJECT_CLASS(klass)         (G_TYPE_CHECK_CLASS_CAST ((klass), GST_TYPE_TEST_OBJECT, GstTestObjectClass))
 
/* 实例结构转换 */
#define GST_TEST_OBJECT_CAST(obj) ((TestObject*)(obj))
 
 
struct _GstTestObject {
  GstObject            object;
  gchar *mem;
  ...
}
 
/* 类定义 */
struct _GstTestObjectClass {
  GstObjectClass    object_class;
  ...
}
```

就是通过上述的宏定义，可以方便的将各种类以及对象进行转换，在子类中可以调用父类的函数等操作，同时，在`gstreamer`中，还有一些属性设置函数等，进行多样化的类管理。

另外的，宏定义`G_DEFINE_TYPE_WITH_CODE`也是实现与`G_DEFINE_TYP`类似的功能，只不过是可以将一些函数内置在`type_name##_get_type()`函数中。
