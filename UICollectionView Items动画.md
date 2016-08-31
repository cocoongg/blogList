# Animating Items in a UICollectionView
原文链接：https://www.raizlabs.com/dev/2014/02/animating-items-in-a-uicollectionview/#Resources

UICollectionView被人们称为新的UITableView一点都不夸张。UIcollectionView在显示上的灵活性以及在与多个
数据项交互上的便利性使得它成为IOS6以后UIKit最重要的工具。UIcollectionView提供了框架使我们能够深度的
定制我们想显示的内容，并且能够实现你能想象到的任何布局，而且，在滚动的时候能够动态的修改每一个view的
显示属性，还支持在独立的布局之间切换，以及插入删除view（或cell）。

在插入删除items的时候，可以定制相应的动画，这在动态用户交互的app时显得非常强大，但是不幸的是，在现有
的资料中，这部分知识很少被提及，也没有很好的文档进行描述。定制此类动画并不复杂，but can be difficult
to figure out from scratch.

下面是本文将要讨论的几个关键点：

+ 在插入删除的时候，cell和supplementary view都可以做动画
- 在插入删除section的时候，无需再插入删除单个的item
* 为了在特定的场景下能够创建合适的动画效果，需要跟踪明白布局失效的处理过程是如何的
+ 在布局失效的处理过程中，可以先计算出来布局属性并且缓存下来
- 最好是完整的创建一个custom layout来更好的控制整个自定义动画

注意：有很多优秀的资源来让我们学习UICollectionView的基础和创建完全自定义的layout，但是本文并不会太深
入的讨论这些。感兴趣的读者可以参考本位最后的Resources一章。

#COLLECTION VIEW 基础

##THE COLLECTION VIEW
*UICollectionView*是*UIScrollView*的子类，因此它支持无限滑动。每个UICollectionView实例必须有一个正确的
数据源(*UICollectionViewDataSource*),数据源为它提供section或者item的个数，以及collection view的显示内容。
另外，还需要有一个布局对象(*UICollectionViewLayout*)来确定这些内容如何显示。UICOllectionView还可以设置一
个代理对象("UICollectionViewDelegate",非必须)，次代理对象用来实现阻止或者响应cell的选中／非选中状态，以及
一些其它的操作。

##THE ITEMS
UICollectionView管理着多种类型的item，所有类型的item都可以做动画。

Cells是UICollectionViewCell的子类，类似于table view的cells，它是显示collection view的内容的最主要的视图对象。
每一个cell描述了一个单独的数据对象，并呈现给用户，例如：一张带有标题的照片。

Cells可以被创建，重用，配置，并且提供给collection view用于显示，这部分内容由collection view的data source提供，
我们需要实现下面的方法：
    - (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
补充视图一般用来提供相应section的一些额外信息，实际上就是替换了table view的header和footer视图的概念。

装饰视图用来为collection view做一些装饰用。例如，为某个section提供一张背景图片，或者在collection view的最后
放置一个装饰用的图标。

补充视图和装饰视图都必须继承自UICollectionReusableView

collection view中所有的item都有一个唯一的indexPath值，这个值用来在collection view中，以及在相应的layout/delegate/
data source中来唯一的确定这个item。但是不同类型的item之间的indexPath值并没有任何关联，甚至可能不同类型的item会有
相同的indexPath值。详见这篇[文档](https://developer.apple.com/library/ios/documentation/uikit/reference/UICollectionViewLayoutAttributes_class/Reference/Reference.html)。
>如何利用indexPath参数来确定一个特定的supplementary视图完全取决于你。一般，可以使用elementKind参数来确定
是何种类型的supplementary视图，然后利用indexPath参数来区分不同的supplementary视图实例。

##THE LAYOUT
Collection view的布局对象是UICollectionViewLayout的子类的实例，它决定了每一个item的布局特性，决定了在特定的
rectangle内哪个item是可见的，并且为每一个item提供一个UICollectionViewLayoutAttributes来确定它的位置，转换，大小
或者其它一些常用属性。

苹果提供了一个具体的子类－UICollectionViewFlowLayout，它根据一些属性设置（横向或者竖直滚动，item间距，section内边距等）
，在collection view的内容区域内均匀的排列一些矩形的item（cell，补充视图，装饰视图）。

布局的时候，可以在item插入，删除的时候定制一些动态的有意思的动画效果。

#插入，删除Items


















