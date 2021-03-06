## SQL语句执行过程
> 当客户端和Server端建立数据库TCP连接以后，客户端的任务就是先Server端发送SQL语句，获得结果。真正的SQL的相关处理发生在Server端

### 当Server端收到了Client发送的SQL语句以后
1. 查询高速缓存，如果存在相同的执行计划，则会直接执行这个SQL省区后面的工作
   * 从内存中读取数据要比从硬盘中的数据文件中读取数据效率要高
   * 另外一方面省去了解析SQL语句
1. 首先对SQL语句进行语法解析，服务器进程就会开始检查这条语句的合法性。这里主要是对 SQL 语句的语法进行检查,看看其是否合乎语法规则。
1. 进行语义解析，服务器进程接下去会对语句中的字段、表等内容进行检查。看看这些字段、表是否在数据库中。
   * 如果表名与列名不准确的话,则数据库会就会反馈错误信息给客户端。
1. 获得对象的解析锁，系统就会对我们需要查询的对象加锁。
   * 这主要是为了保障数据的一致性,防止我们在查询的过程中,其他用户对这个对象发生改变。
1. 数据访问权限的核对，服务器进程还会检查,你所连接的用户是否有这个数据访问的权限。若你连接上服务器的用户不具有数据访问权限的话,则客户端就不能够取得这些数据。
   * 数据库服务器进程**先检查语法与语义,然后才会检查访问权限**
1. 确定最佳执行计划。服务器进程会根据一定的规则,对这条语句进行优化
   * 不过要注意,这个优化是有限的。
   * 一般在应用软件开发的过程中,需要对数据库的 sql 语言进行优化,这个优化的作用要大大地大于服务器进程的自我优化。
   * 当服务器进程的优化器**确定这条查询语句的最佳执行计划后,就会将这条 SQL 语句与执行计划保存到数据高速缓存(library cache)**。如此的话,等以后还有这个查询时,就会省略以上的语法、语义与权限检查的步骤,
而直接执行 SQL 语句,提高 SQL 语句处理效率。
1. 执行语句。执行语句也分成两种情况
   * 一是若被选择行所在的数据块已经被读取到数据缓冲区的话,则服务器进程会直接把这个数据传递给客户端,而不是从数据库文件中去查询数据。
   * 若数据不在缓冲区中,则服务器进程将从数据库文件中查询相关数据,并把这些数据放入到数据缓冲区中(buffer cache)。

1. 提取结果。当语句执行完成之后,查询到的数据还是在服务器进程中,还没有被传送到客户端的用户进程，所以,在服务器端的进程中,有一个专门负责数据提取的一段代码。他的作用就是把查询到的数据结果返回给用户端进程,从而完成整个查询动作。从这整个查询处理过程中,我们在数据库开发或者应用软件开发过
程中,需要注意以下几点:
   * 一是要了解数据库缓存跟应用软件缓存是两码事情。数据库缓存只有在数据库服务器端才存在,在
客户端是不存在的。只有如此,才能够保证数据库缓存中的内容跟数据库文件的内容一致。才能够根据
相关的规则,防止数据脏读、错读的发生。而应用软件所涉及的数据缓存,由于跟数据库缓存不是一码事
情,所以,应用软件的数据缓存虽然可以提高数据的查询效率,但是,却打破了数据一致性的要求,有时候
会发生脏读、错读等情况的发生。所以,有时候,在应用软件上有专门一个功能,用来在必要的时候清除
数据缓存。不过,这个数据缓存的清除,也只是清除本机上的数据缓存,或者说,只是清除这个应用程序
的数据缓存,而不会清除数据库的数据缓存。
   * 二是绝大部分 SQL 语句都是按照这个处理过程处理的。我们 DBA 或者基于 Oracle 数据库的
开发人员了解这些语句的处理过程,对于我们进行涉及到 SQL 语句的开发与调试,是非常有帮助的。有
时候,掌握这些处理原则,可以减少我们排错的时间。特别要注意,数据库是把数据查询权限的审查放在
语法语义的后面进行检查的。所以,有时会若光用数据库的权限控制原则,可能还不能满足应用软件权限
控制的需要。此时,就需要应用软件的前台设置,实现权限管理的要求。而且,有时应用数据库的权限管
理,也有点显得繁琐,会增加服务器处理的工作量。因此,对于记录、字段等的查询权限控制,大部分程
序涉及人员喜欢在应用程序中实现,而不是在数据库上实现。

---

### SQL语句中的函数、关键字、排序等执行顺序:
1. FROM 子句返回初始结果集。
2. WHERE 子句排除不满足搜索条件的行。
3. GROUP BY 子句将选定的行收集到 GROUP BY 子句中各个唯一值的组中。
4. 选择列表中指定的聚合函数可以计算各组的汇总值。
5. 此外,HAVING 子句排除不满足搜索条件的行。
6. 计算所有的表达式;
7. 使用 order by 对结果集进行排序。
8. 查找你要搜索的字段。