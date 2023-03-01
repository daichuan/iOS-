#iOS笔记/复习笔记
---
1. Category的实现原理？
原理：底层结构是结构体 categoty_t 创建好分类之后分两个阶段：
	1. 编译阶段：
将每一个分类都生成所对应的 category_t结构体， 结构体中存放分类的所属类name、class、对象方法列表、类方法列表、协议列表、属性列表。

	2. Runtime运行时阶段：
将生成的分类数据合并到原始的类中去，某个类的分类数据会在合并到一个大的数组当中（后参与编译的分类会在数组的前面），分类的方法列表，属性列表，协议列表等都放在二维数组当中，然后重新组织类中的方法，将每一个分类对应的列表的合并到原始类的列表中。（合并前会根据二维数组的数量扩充原始类的列表，然后将分类的列表放入前面）

2. 为什么Category的中的方法会优先调用？
如上所述， 在扩充数组的时候 会将原始类中拥有的方法列表移动到后面， 将分类的方法列表数据放在前面，所以分类的数据会优先调用

3. 延伸问题 - 如果多个分类中都实现了同一个方法，那么在调用该方法的时候会优先调用哪一个方法？
在多个分类中拥有相同的方法的时候， 会根据编译的先后顺序 来添加分类方法列表， 后编译的分类方法在最前面，所以要看 Build Phases —> compile Sources中的顺序。 后参加编译的在前面。

4. 扩展和分类的区别
扩展@interface 是匿名分类， 不是分类。 就是属性添加 在编译的时候就加入到了类中
category在runtime中才合并的。

5. Category的实现原理，以及Category为什么只能加方法不能加属性?
分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。


```c
struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods; // 对象方法
    struct method_list_t *classMethods; // 类方法
    struct protocol_list_t *protocols; // 协议
    struct property_list_t *instanceProperties; // 属性
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};

```



参考链接
[iOS底层原理总结 — 利用Runtime源码 分析Category的底层实现](https://juejin.cn/post/6844903992682610695)
[iOS底层原理总结 - Category的本质 - 简书](https://www.jianshu.com/p/fa66c8be42a2)