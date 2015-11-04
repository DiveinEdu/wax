Wax现在由阿里巴巴维护
------------------
感谢@probablycorey创建了这么一个伟大的项目。
Wax是连接Lua和Objective-C最好的桥梁，我们将在这里继续维护它。我们已经修复了许多问题，诸如64位支持和线程安全，同时也添加了许多新的特性，比如Lua函数和OC的Block之间相互转换、在Lua中调用OC的Block、获取和修改私有变量、内置常用的C函数以及Lua代码调试。

Wax
---
Wax是一个能够让我们在iPhone上使用[Lua](http://www.lua.org/about.html)的框架。它使用Objective-C的运行时桥接了Objective-C和Lua。利用Wax，我们可以在Lua中使用任何在Objective-C中所能使用的功能。还在等什么呢，赶紧试一下！

为什么要用Lua编写iPhone应用？
-------------------------
我喜欢写iPhone应用，但是如果有一门比Objective-C更加动态的语言会更好。下面是为什么许多人都喜欢用Lua + Wax而不是Objective-C的一些原因...

* 自动内存管理（GC）！而不是`alloc`、`retain`和`release`。
* 写更少的代码！不再有头文件、静态类型、数组、字典等！Lua可以让我们用更少的代码做更多的事情。
* 可以访问任何Cocoa、UITouch、Foundation等框架中的任何一个，哪怕它们是用Objective-C编写的。Wax将这些东西都自动暴露给了Lua。我们可以使用任何一个喜欢的框架。
* 超级简单的HTTP请求。与REST webservice的交互简单得不能再简单。
* Lua有闭包，也就是Objective-C中的Block！任何用过的人都知道它们到底有多么强大。
* Lua有一个内置的正则表达式库。

示例
---
在[examples folder](https://github.com/alibaba/wax/tree/master/examples)中有一些简单的Wax应用。

如何创建一个`UIView`并且将颜色设为红色？

``` lua
-- 忘记alloc吧，Wax中的内存是自动管理的
view = UIView::initWithFrame(CGRect(0, 0, 320, 100))

-- 使用冒号给Objective-C对象发送消息
-- 可以使用同样的方式访问UIView中的任何方法
view:setBackgroundColor(UIColor:redColor)
```

多个参数的方法怎么办？

``` lua
-- 使用下划线(_)连接方法名（Objective-C中的标号），然后像C语言的函数一样调用就可以了。
UIApplication:sharedApplication():setStatusBarHidden_animated(true, false)
```

怎样通过消息发送数组、字符串或者字典？

``` lua
-- Wax自动将数组、字符串和字典对象转换为NSArray、NSString、NSDictionary对象
images ＝ {"myFace.png", "yourFace.png", "theirFace.png"}
imageView = UIImageView:initWithFrame(CGRect(0, 0, 320, 460))
imageView:setAnimationImages(images)
```

如何创建一个自定义的`UIViewController`？

``` lua
-- 在"MyController.lua"中创建
--
-- 下面代码创建了一个继承自UIViewController的Objective-C类"MyController"，并且能够在Objective-C中使用。
waxClass{"MyController", UIViewController}

function init()
  -- 使用self.super调用父类的方法
  self.super:initWithNibName_bundle("MyControllerView.xib", nil)
  return self
end

function viewDidLoad()
  -- 自定义代码
end
```

不是说HTTP请求非常简单嘛，我不太相信...

``` lua
url = "http://search.twitter.com/trends/current.json"

-- 创建一个异步调用，在接收到响应时调用回调函数
wax.http.request{url, callback = function(body, response)
  -- response是一个NSHTTPURLResponse对象
  puts(response:statusCode())

  -- 如果Content-Type是JSON，Wax会自动解析并放入一个Lua table中
  puts(body)
end}
```

Wax将`NSString`、`NSArray`、`NSDictionary`和`NSNumber`转化为原生的Lua值，但有时又需要将Lua对象转化成Objective-C对象。下面是一个例子。

``` lua
local testString = "Hello lua!"
local bigFont = UIFont:boldSystemFontOfSize(30)
local size = toobjc(testString):sizeWithFont(bigFont)
puts(size)
```

如何将Lua的函数转换为Objective-C的Block？[更多详情](https://github.com/alibaba/wax/wiki/Block)。

``` lua
UIView:animateWithDuration_animations_completion(1,
  toblock(
    function()
      label:setCenter(CGPoint(300, 300))
    end
  ),
  toblock(
    function(finished)
      print('lua动画执行完成' .. tostring(finished))
    end
    ,{"void", "BOOL"}))
)

--- OC的方法是：-(void)testReturnIdWithFirstIdBlock:(id(^)(id aFirstId, BOOL aBOOL, int aInt, NSInteger aInteger, float aFloat, CGFloat aCGFloat, id aId))block
self:testReturnIdWithFirstIdBlock(
  toblock(
    function(aFirstId, aBOOL, aInt, aInteger, aFloat, aCGFloat, aId)
      print("aFirstId=" .. tostring(aFirstId))
      -- assert(aFirstId == self, "aFirstId不等")
      assertResult(self, aBOOL, aInt, aInteger, aFloat, aCGFloat, aId)
      print("LUA测试成功: testReturnIdWithFirstIdBlock")
      return aFirstId
    end
    , {"id", "id", "BOOL", "int", "NSInteger", "float", "CGFloat", "id"}
  )
)
```

如何调用Objective-C的Block？

``` lua
-- OC的Block类型是 id (^)(NSInteger, id, BOOL, CGFloat)
local res = luaCallBlock(block, 123456, aObject, true, 123.456)

-- 或者
local res = luaCallBlockWithParamsTypeArray(block, {"id", "NSInteger", "id", "BOOL", "CGFloat"}, 123456, aObject, true, 123.456)
```

如果实例变量是私有的，怎么办？

``` lua
print(self:view():getIvarInt("_countOfMotionEffectsInSubtree"))

self:setIvar_withObject("_title", "abcdefg")
print(self:getIvarObject("_title"))

self:setIvar_withObject("_infoDict", {k11="v11", k22="v22"})
print(toobjc(self:getIvarObject("_infoDict")))
```

如何调用C函数？

``` lua
luaSetWaxConfig({wax_openBindOCFunction=true}) --绑定系统的C函数

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0),
  toblock(
    function()
      print(string.format("dispatch_async"))
    end
  )
);

UIGraphicsBeginImageContext(CGSize(200, 400))
UIApplication:sharedApplication():keyWindow():layer:renderInContext(UIGraphicsGetCurrentContext())
local aImage = UIGraphicsGetImageFromCurrentImageContext()
UIGraphicsEndImageContext()

local imageData = UIImagePNGRepresentation(aImage)
print("imageData.length = ", imageData.length)
local image = aImage
local data = UIImageJPEGRepresentation(image, 0.8)
print("data.length = ", data:length())
```

Lua代码调试
---------
有什么办法可以调试Lua代码吗？
当然，你可以使用强大的ZeroBraneStudio进行调试。[更多详情](https://github.com/alibaba/wax/tree/master/examples/LuaCodeDebug)。

Watch OS
--------
Wax可以在watch OS上运行吗？
感谢Lua的跨平台特性，Wax当然可以在watch OS上运行。见`tools/WaxWatchFramework and examples/WaxWatchExample`。

使用cocoapods
------------
见`examples/TestWaxPod`。
* 在Podfile中添加`pod 'wax', :git=>'git@github.com:alibaba/wax.git', :tag=>'1.1.0'`。（可以使用任意你所需要的tag）
* 运行lua代码。

``` lua
wax_start(nil, nil);
wax_runLuaString("print('hello wax')");
```

设置&教程
--------

[设置Wax](https://github.com/alibaba/wax/wiki/Installation)

[Wax如何工作?](https://github.com/alibaba/wax/wiki/Overview)

[Wax写的简单Twitter客户端](https://github.com/alibaba/wax/wiki/Twitter-example)

包含哪些API？
-----------

所有的API！任何用Objective-C编写的代码（包括外部框架）都能在Wax中工作！UIAcceleration、MapKit、GameKit都正常运行。

作者
---
Corey Johnson (probablycorey at gmail dot com)

更多
---
* [Feature Requests? Bugs?](https://github.com/alibaba/wax/issues) - Issue tracking and release planning.
* [Mailing List](http://groups.google.com/group/iphonewax)
* Quick questions or issues? Send an email to [@Zhengwei Yin (Junzhan)](mailto:junzhan.yzw@taobao.com)

贡献者
-----
[@Zhengwei Yin (Junzhan)](mailto:junzhan.yzw@taobao.com)  
Fork it, change it, commit it, push it, send pull request; instant glory!


The MIT License
---------------
Wax is Copyright (C) 2009 Corey Johnson See the file LICENSE for information of licensing and distribution.
