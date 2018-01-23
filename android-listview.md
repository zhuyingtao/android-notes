#### ListView

![img](http://img.blog.csdn.net/20150704111744498?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### Adapter

Adapter是适配器的意思，它在ListView和数据源之间起到了一个桥梁的作用，ListView并不会直接和数据源打交道，而是会借助Adapter这个桥梁来去访问真正的数据源。

![img](http://img.blog.csdn.net/20150719202924532?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### RecycleBin机制

它是写在AbsListView中的一个内部类，所以所有继承自AbsListView的子类，也就是ListView和GridView，都可以使用这个机制。这个机制也是ListView能够实现成百上千条数据都不会OOM最重要的一个原因。

![img](http://img.blog.csdn.net/20150719213754421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)





