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

##COLLECTION VIEW
*UICollectionView*是*UIScrollView*的子类，因此它支持无限滑动。每个UICollectionView实例必须有一个正确的
数据源(*UICollectionViewDataSource*),数据源为它提供section或者item的个数，以及collection view的显示内容。
另外，还需要有一个布局对象(*UICollectionViewLayout*)来确定这些内容如何显示。UICOllectionView还可以设置一
个代理对象("UICollectionViewDelegate",非必须)，次代理对象用来实现阻止或者响应cell的选中／非选中状态，以及
一些其它的操作。

##ITEMS
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

##布局
Collection view的布局对象是UICollectionViewLayout的子类的实例，它决定了每一个item的布局特性，决定了在特定的
rectangle内哪个item是可见的，并且为每一个item提供一个UICollectionViewLayoutAttributes来确定它的位置，转换，大小
或者其它一些常用属性。

苹果提供了一个具体的子类－UICollectionViewFlowLayout，它根据一些属性设置（横向或者竖直滚动，item间距，section内边距等）
，在collection view的内容区域内均匀的排列一些矩形的item（cell，补充视图，装饰视图）。

布局的时候，可以在item插入，删除的时候定制一些动态的有意思的动画效果。

#插入，删除Items
UICollectionView提供了几个方法以动画的形式来更新cell的位置和内容，而不是简单的调用reloadData使它立刻更新。这样，当
有新的内容要显示或者响应用户的操作（例如删除照片）时能够给用户更多的视觉上的反馈。
    // Insert items

    - (void)insertItemsAtIndexPaths:(NSArray *)indexPaths;

    // Insert sections

    - (void)insertSections:(NSIndexSet *)sections;

    // Remove items

    - (void)deleteItemsAtIndexPaths:(NSArray *)indexPaths;

    // Remove sections

     - (void)deleteSections:(NSIndexSet *)sections;

    // Move item

    - (void)moveItemAtIndexPath:(NSIndexPath *)indexPath toIndexPath:(NSIndexPath *)newIndexPath;

    // Reload items

    - (void)reloadItemsAtIndexPaths:(NSArray *)indexPaths;
##BATCH UPDATES
UICollectionView还提供了基于block的方法来同步cell动画，减少在执行一组更新的时候的计算上的开销：
    - (void)performBatchUpdates:(void (^)(void))updates completion:(void (^)(BOOL finished))completion;
在updates这个block中，任何一个item的 inset/remove/move 调用都会被预先计算好，并一起同步执行。当动画停止以后，completion
这个block被执行，参数finished表示动画是被打断了还是执行完成了。对于有UIView动画经验的读者，这部分应该比较熟悉了。

##警告
在执行batched updates的时候，有一些重要的点需要注意，其中一些在苹果的文档上并没有明确的说明：

1.和UITableView一样，在执行动画之前，collection view的data source必须有正确的item/section个数来适应这些修改。
2.在batch upate这个block中插入一个新的section的时候，一定不要插入任何item到新的section，这些item会被系统隐式的插入。
  事实上，在batch update中显示的插入item到新创建的section中将会在collection view的layer树中创建一些幽灵layer。这可能是
  UICollectionView的一个bug，因为文档中没有对此进行说明，也没有抛出任何异常。
3.在batch update中删除一个section的时候，不需要先删除它的item，虽然这样做不会导致任何不好结果，但是这完全是多余的。
4.虽然文档中没有说明，但是在batch update中调用reloadItemsAtIndexPaths:是很不安全的。因为，如果在这个block中还执行了
  一些其它的操作，而这些操作也导致了这个被reload的Item产生一些替换或者其它形式的动画，就将会抛出异常，因为对于同一个Item
  有多个动画。

##补充视图的一些说明
当插入，删除的时候，补充视图也可以做一些动画，但是这仅限于这些插入，删除的动作是因为布局或者data source的改变导致的。
通过在layoutAttributesForItemsInRect:方法中返回当前可见的每一个补充视图的layout attributes，layout负责来确定补充视图
的个数。data source通过collectionView:viewForSupplementaryElementOfKind:atIndexPath:方法负责创建和返回相应的补充视图。

因此，如果不是显式的设置layout无效或者reload data source，补充视图是不能动态的插入删除的。设置layout无效之前的补充视图
的布局属性如果和设置layout无效之后的布局属性有任何的不同，都会被做成适当的动画展示出来。

UICollectionViewFlowLayout有内置的补充视图，每一个section都有一个header视图和一个footer视图。在collectionView:viewForSupplementaryElementOfKind:atIndexPath:方法中，data source可以返回一个相应的视图（header，footer），或者返回nil来隐藏补充视图。当
以动画的形式插入或者删除一个section的时候，其相关的header/footer也会以动画的形式被展现。

##在布局中定制动画







































