## 与知识的联结
这里主要说的其实是分层，分层让我们只用关注自己层面的事情而不用要从底层开始一步一步了解整个系统架构。

正因为有了网络的七层架构，所以每一层都只负责和相邻的层通信，而不关心其他层。
所以我们现在写代码只用关系http调用相关的知识，而不必去一点点了解tcp，udp等等工作原理。

分层的核心就在于每一层可以只关心自己层的事情，这需要每一层都有良好的抽象。层和层之间通过一些转换来和各层的抽象关联。