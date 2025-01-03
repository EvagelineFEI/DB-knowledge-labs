# 23fall-lab0

## copy-on-write说明

每年的lab目测都会有些不一样，特别是lab0的主题都会不一样。

23fall的lab0是实现一个copy-on-write的tire（字典树），支持查找，修改，删除操作；以及对这些操作的并行化。

这个实验的核心是让学生了解”copy-on-write“，这是DBMS中一个很重要的思想和方法，很多DBMS核心技术的实现，比如MVCC（多版本并发控制），都有”copy-on-write“的思想，可以提升DBMS的效率和支持多用户的能力；而tire只是lab0选择的一种数据结构，存储数据的结构还可以是B-tree等。

所谓”copy-on-write“写时复制，是为了保证多用户访问时，通过不加锁的方式实现数据完整性，支持支持读-读并行和读-写并行；键都用边表示，值则存在内存空间中：

例如：

<img src="figures/lab0-c-o-w-intro.png" width="100%" height="auto" />

原本内存中有键值对：("ab", 1) 和 ("ac", "val") 如图1。

如果要插入一个新的键值对("ad", 2)。做法是复制一个tire页，然后在这个新的tire页上加；发现”ab“和”ac“对应的值没有变，于是ab“和”ac“指向原来tire页的内存空间；而对于新增的值，则会开辟新的存储空间存放。如图2。

这时候又要插入一个新的键值对 ("b", 3)。一样也是复制一个新的tire页，发现”a“对应的值都没有变，于是”a“指向原来的内存空间。新增的("b", 3)则开辟新的边和新的空间存放。

<img src="figures/lab0-c-o-w-intro_.png" width="40%" height="auto" />

如果这时候插入 ("a", "abc")，删除（"ab", 1)，我们会得到图4的模样。

可以想象一下过程：同样是复制一个新的tire页，插入 ("a", "abc")的时候发现尽管有("ab", 1) ，("ac", "val") 和("ad", 2)，但是a指向的那块内存空间是空的可以用，于是新的tire页的“a”指向上一个tire页的那块空间，并在里面放入“abc”。

要删除（"ab", 1)，那么顺着“a”边出发，找到“b”边和“b”指向的空间里面那块存储1的内存，把这块内存回收后，删除“b”边。

想象有两个用户A和B。

用户A想要插入("ad", 2)的同时，用户B想要读取“ac”。那么用户B读到的就是原来副本里的数据。在并发读写数据库时，copy-on-write可以做到在读操作时不用阻塞写操作，写操作也不用阻塞读操作，提高了数据库并发读写的性能。

## 实现-感悟