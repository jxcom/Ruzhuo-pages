[Home](https://wecache.com)

# UIKit 小细节 ——Label 的对齐方式

这种很忙，也很沮丧，压力很大。

但既然说了每周更新一些东西，哪怕只是只言片语，也应该写写。就更新一个很小的细节吧，今天刚在 kodeco 上看到的。主要是讲了一些关于控件对齐的事情，关于控件对齐的问题居然还有一本书专门去讲：

https://www.kodeco.com/books/auto-layout-by-tutorials

今天遇到什么问题呢，主要是当 Label 内容换行的时候，它对齐的方式居然是向左下角对齐，而不是左上角，我不清楚什么时候修改了，还是默认如此，一行的情况下是正常：



多行的情况下 Principals 出现漂移：



询问 ChatGPT，得到的答案是可能是 constraint 的问题，发现 Principals 这个 Label 的一个 constraint 居然是 bottom，怀疑是这个有关改成 top ，好了：



后来也出现了几个对齐的问题，大都与 constraint 设置有关，看来这个东西虽然可以帮助我们更方便的对齐，但也要小心暗藏一些小动作。

今天玩 UIKit 玩出了当年 MFC 摆控件的感觉。

