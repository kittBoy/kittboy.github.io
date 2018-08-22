---
"title": "ckeditor引导程序"
"date": "2016/7/7 20:46:25"
categories:
- js
- ckeditor

tags:
- js
- ckeditor
---

### 前言
最近公司的老大让搞一下富文本编辑器ckeditor,要兼容老数据。所以就研究了下ckeditor,写几篇文章记录下学习成果。

### ckeditor-bootstrap
1. 入口文件/ckeditor-dev/ckeditor.js -> /ckeditor-dev/core/ckeditor_base.js

- 创建唯一的全局CKEDITOR对象
- 启动加载函数->/core/loader.js,并指定默认加载函数为ckeditor
- 设置ckedtor状态

   unload the API is not yet loaded.

   basic_loaded: the basic API features are available.

   basic_ready: the basic API is ready to load the full core code.

   loaded**: the API can be fully used.


2. loader.js

 - 加载ckeditor文件，并在加载其之前加载依赖文件。

3. event.js

- 各模块通过继承event的原型链实现事件机制。

4. editor_basic.js

- 通过继承event对象，实现事件机制，并覆盖默认的事件触发方法 fire和fireOnce

5. env.js.检测浏览器环境，并将其保存在CKEDITOR.env对象中。

6. ckeditor_basic.js CKEDITOR通过继承event实现事件机制； 绑定dom加载完成事件，在dom加载完成后，调用onload函数加载full code;并设置当前状态为basic_loaded;
7. log.js 绑定函数到CKEDITOR对象，通过触发log事件，实现信息的打印；并为CKEDITOR绑定log事件。
8. dom.js 初始化dom对象；
9. tools.js 初始化CKEDITOR.tools对象；
10. dtd.js 通过定义ckeditor.dtd对象，实现ckeditor内部标签嵌套规范。
11. dom-event.js 封装dom事件接口，屏蔽各浏览器差异。
12. dom-domobject.js 封装原生dom事件接口。
13. dom-node.js 封装原生dom操作屏蔽各浏览器实现差异，定义统一操作接口。
14. dom-window.js 封装原生window对象
15. dom-document.js 封装原生document对象
16. dom-nodelist 封装原生nodelist对象。
17. dom-element.js 封装原生element操作，屏蔽浏览器差异，提供统一对外接口。
18. dom-documentfragment.js 封装原生documetnfragment操作，屏蔽浏览器差异，提供统一对外接口。
19. dom-walker.js 提供了遍历selection对象的方法。是ckeditor过滤机制底层实现的一部分。
20. dom-range.js 封装了原始的range操作，屏蔽了浏览器差异，提供统一对外接口。
21. command.js ckeditor插件的核心实现的一部分，提供了ckeditor的命令机制。
22. ckeditor_base.js
23. config.js ckeditor默认配置
24. filter.js ckeditor 过滤机制的底层实现，是allowedContent和disallowedContent的底层支持
25. focusmanager.js 封装了focus事件，提供了锁机制。
26. keystrokehandler.js 将键盘事件绑定到ckeditor对象上。
27. lang.js ckeditor 多语言支持。
28. scriptloader.js 实现了异步加载js队列。
29. resourcemanager.js ckeditor资源管理器，是插件机制的底层支持之一。
30. plugins.js 加载插件。
31. ui.js ckeditor界面控制器。
32. editor.js 代表了编辑器实例，实现了事件机制，加载客户端配置，初始化编辑器实例，包括界面，插件，事件等。
33. htmlparser.js 解析html.
34. htmlparser-basicwriter.js 提供了文本到html的转换
35. htmlparser-node.js 提供了解析html时基本的node操作
36. htmlparser-comment.js 提供了解析html时对comment的过滤和插入。
37. htmlparser-text.js 解析html时过滤和插入的支持
38. htmlparser-cdata.js 提供了解析html时过滤和插入cdata的支持
39. htmlparser-fragment.js 提过了解析html时对fragment内容的过滤和插入的支持
40. htmlparser-filter.js 封装了底层过滤机制，供上层调用。
41. htmldataprocessor.js 组合htmlparser功能，通过事件机制，对文本到editor的输入和输出进行转化。
42. htmlparser-element.js 提供了解析html时过滤和插入element对象的属性和样式的支持
43. template.js 提供了模板样式的支持
44. ckeditor.js 实例化ckeditor.document对象，加载_bootstrap。
45. creators-inline.js 获取ckeditor实例dom元素，并将其contenteditable设置为true.
46. creators-themedui.js 替换textarea为ckeditor instances.
47. editable.js 可编辑区域的交互API实现
48. selection.js 鼠标区域选择API实现
49. style.js ckeditor自定义样式API封装，挂载点 CKEDITOR.style.
50. dom-comment.js 封装原生的添加注释方法，屏蔽浏览器差异，提供统一API.
51. dom-elementpath.js 提供获得dom节点路径的API.
52. dom-text.js 封装元素text的操作API。
53. dom-ranglist.js 封装对ranglist集合的操作API.
54. skin.js ckeditor界面皮肤设置。
55. _bootstrap.js ckeditor初始化完成，用户代码可以执行。