title: 通过runtime转换对象为字典

tags:

- runtime
  
- NSDictionary
  
  categories:
  
- Object-C
  
  ​
  
  date: 2015-11-12 15:34:00

---

### 一，[runtime](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html)的一些小知识

众所周知，[Objective-C](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html)是一种面向对象的高级动态语言，它可以将很多编译时期做的事情放到运行时来处理，比如创建类和对象、进行消息传递和转发等，使得代码更具灵活性。与Object-C runtime 交互有三种方式：Objective-C源代码、NSObject方法及runtime的函数。

### 二，动态获取属性名称

首先介绍一下runtime中的property，它是一个指向`objc_property`结构体的指针：

``` objc
// An opaque type that represents an Objective-C declared property.
typedef struct objc_property *objc_property_t;
```

可以通过`class_copyPropertyList` 和 `protocol_copyPropertyList`方法来获取类和协议中的属性：

``` objc
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```

比如获取某个类（obj）的属性列表：

``` objc
unsigned int propsCount;
objc_property_t *props = class_copyPropertyList([obj class], &propsCount)
```

通过property_getName方法就可以得到某个类属性的名字了：

``` objc
unsigned int propsCount;
objc_property_t *props = class_copyPropertyList([obj class], &propsCount);
for(int i = 0;i < propsCount; i++)
{
    objc_property_t prop = props[i];
    NSString *propName = [NSString stringWithUTF8String:property_getName(prop)];
}
```

完整代码如下：

``` objc
+ (NSDictionary*)getDicFomObj:(id)obj
{
    NSMutableDictionary *dic = [NSMutableDictionary dictionary];
    unsigned int propsCount;
    objc_property_t *props = class_copyPropertyList([obj class], &propsCount);
    for(int i = 0;i < propsCount; i++)
    {
        objc_property_t prop = props[i];

        NSString *propName = [NSString stringWithUTF8String:property_getName(prop)];
        id value = [obj valueForKey:propName];
        if(value == nil)
        {
            value = [NSNull null];
        }
        else
        {
            value = [self getObjectInternal:value];
        }
        [dic setObject:value forKey:propName];
    }
    return dic;
}
```



### 三，解析属性对应的数据结构

得到类属性的名称后，就可以知道该属性对应的类型了，如果是Object-C class，直接判断数据类型即可，比如NSString、NSArray、NSDictionary等。如果该属性的值对应的是派生类，则需要回到上一步重新解析，直到遍历完为止。

``` objectivec
+ (id)getObjectInternal:(id)obj
{
    if([obj isKindOfClass:[NSString class]]
       || [obj isKindOfClass:[NSNumber class]]
       || [obj isKindOfClass:[NSNull class]])
    {
        return obj;
    }

    if([obj isKindOfClass:[NSArray class]])
    {
        NSArray *objarr = obj;
        NSMutableArray *arr = [NSMutableArray arrayWithCapacity:objarr.count];
        for(int i = 0;i < objarr.count; i++)
        {
            [arr setObject:[self getObjectInternal:[objarr objectAtIndex:i]] atIndexedSubscript:i];
        }
        return arr;
    }

    if([obj isKindOfClass:[NSDictionary class]])
    {
        NSDictionary *objdic = obj;
        NSMutableDictionary *dic = [NSMutableDictionary dictionaryWithCapacity:[objdic count]];
        for(NSString *key in objdic.allKeys)
        {
            [dic setObject:[self getObjectInternal:[objdic objectForKey:key]] forKey:key];
        }
        return dic;
    }

    // 当属性值为派生类时，重头解析
    return [self getDicFomObj:obj];
}
```

如此以来，通过getDicFomObj与getObjectInternal两个方法，即可完成多重结构的类的解析。

参考：http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/