Android实现Tab类型的界面方式：

- 传统的ViewPager实现

主要就是ViewPager+ViewAdapter。

评价：所有的代码都集中在一个Activity中，显得代码比较乱。

- FragmentManager+Fragment实现

主要利用了Fragment在主内容界面对Fragment的add,hide等事务操作。

评价：每个Fragment中的控件的处理，都是独立到各自的类中，相对来说主Activity简化了不少，可惜没有左右滑动的效果了。

- ViewPager+FragmentPagerAdapter实现

主要通过ViewPager和FragmentPagerAdapter一起来实现。

评价：实现效果和第一种效果一模一样，每个Fragment独自处理自己内部的逻辑，代码整洁很多，并且支持左右滑动。感觉是第一种和第二种的结合版本。

- TabPageIndicator+ViewPager+FragmentPagerAdapter

实现方式和3是一致的，但是使用了TabPageIndicator作为tab的指示器，效果还是不错的