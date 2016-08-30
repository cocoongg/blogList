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

