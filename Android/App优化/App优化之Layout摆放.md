## App优化之Layout摆放

不论是什么样的优化，前期的工作肯定是分析占得时间更多，只有找到问题所在才能对症下药。

我们在优化布局的时候可以使用到以下工具来帮助我们，比如HierarchyViewer和Lint。

HierarchyViewer的使用见该篇文章[App优化之HierarchyViewer的使用]()。

Lint我们平常在做代码检查的时候一般也会用到，只需要检查当前的xml文件就行了，因此相对简单。

### 1. 布局优化的建议

- 尽量不要嵌套使用RelativeLayout
- 尽量不要在嵌套的LinearLayout中都使用weight属性
- 布局容器的选择应以减少View树层级为主
- 去除不必要的父布局
- 善用TextView的Drawable减少布局层级
- 如果HierarchyViewer查看的除去系统的必要层级外超过5层(个人建议，具体视情况来定)就需要考虑优化布局了
- 多利用<include/>，<merge />，<ViewStub/>标签来增加View的复用以及减少层级

