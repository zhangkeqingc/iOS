# IOS数据库篇之realm数据库详解



# 一：realm介绍

Realm是由Y Combinator公司孵化出来的一款可以用于iOS(同样适用于Swift&Objective-C)和Android的跨平台移动数据库。历经几年才打造出来，为了彻底解决性能问题，核心数据引擎用C++打造，并不是建立在SQLite之上的ORM，所以Realm相比SQLite和CoreData而言更快、更好、更容易去使用和完成数据库的操作花费更少的代码。它旨在取代CoredData和sqlite,它不是对coreData的简单封装、相反的，Realm它使用了它自己的一套持久化存储引擎。而且Realm是完全免费的，这不仅让它变得更加的流行也使开发者使用起来没有任何限制。

Realm是一个类MVCC数据库，每个连接的线程在特定的时刻都有一个数据库的快照。MVCC（Multi-Version Concurrent Control 多版本并发控制）在设计上采用了和Git一样的源文件管理算法，也就是说你的每个连接线程就好比在一个分支(也就是数据库的快照)上工作，但是你并没有得到一个完整的数据库拷贝。Realm和一些真正的MVCC数据库如MySQL是不同的，Real在某个时刻只能有一个写操作，且总是操作最新的数据版本，不能在老版本操作。



![img](https:////upload-images.jianshu.io/upload_images/5210585-d8ae9c0e8b6e16ff.png?)

Realm数据库文件管理图示                                        

**Realm数据库使用了零拷贝技术，这是与CoreData及其他数据库完全不同的地方。**

通常的数据库操作是这样的，数据存储在磁盘的数据库文件中，我们的查询请求会转换为一系列的SQL语句，创建一个数据库连接。数据库服务器收到请求，通过解析器对SQL语句进行词法和语法语义分析，然后通过查询优化器对SQL语句进行优化，优化完成执行对应的查询，读取磁盘的数据库文件(有索引则先读索引)，返回对应的数据内容并存储到内存中，数据还需要序列化成内存可存储的格式，最后数据还要转换成语言层面的类型，比如Objective-C的对象等。

而Realm完全不同，它的数据库文件是通过memory-mapped，也就是说数据库文件本身是映射到内存中的，Realm访问文件偏移就好比文件已经在内存中一样(这里的内存是指虚拟内存)，它允许文件在没有做反序列化的情况下直接从内存读取，提高了读取效率。

## 二：Realm的特点

Realm以难以令人置信的快速和易用让开发者能够用仅仅几行代码完成你所需要的一切功能。它旨在打造让用户得在到移动领域离线时的最好体验，我整理了Realm具有的如下特点:

易安装：正如你在将要看到的使用Realm工作。安装Realm就像你想象中一样简单。在Cocoapods中使用简单命令，你就可以使用Realm工作。

速度上：Realm是令人无法想象的快速使用数据库工作的库。Realm比SQLite和CoreData更快，这里的数据就是最好的证明。

跨平台：Realm数据库文件能够跨平台和可以同时在iOS和Andriod使用。无论你是使用Java, Objective-C, or Swift，你都可以使用你的高级模型。

可扩展性：在开发你的移动App特别是如果你的应用程序涉及到大量的用户和大量的记录时，具有良好的可扩展性是非常重要的。

好的文档&支持：Realm团队提供了可读的，非常有组织的并且丰富的文档。如果你遇到什么问题通过Twitter、GitHub或者Stackoverflow与他们交流。（中文api文档地址 https://realm.io/cn/docs/objc/latest/#primary-keys）

可靠性：Realm已经被巨头公司使用在他们的移动App中，像Pinterest, Dubsmash, and Hipmunk。

免费性：使用Realm的所有功能都是免费的。

懒加载：只有当你真正访问对象的值时候才真正从磁盘中加载进来。

## 三：realm与其他数据库的效率比较



![img](https:////upload-images.jianshu.io/upload_images/5210585-44e3e7a7b756ce13.png?)

realm_benchmark_counts                                        



![img](https:////upload-images.jianshu.io/upload_images/5210585-8b15d951201c0212.png?)

realm_benchmark_query                                        



![img](https:////upload-images.jianshu.io/upload_images/5210585-6d3e9d4bed901f4e.png?)

realm_benchmark_insert                                        

Realm的消息通知、数据加密、JSON支持等特性让Realm直接区别于SQLite和CoreData。也为我们切换到Realm提供了理由支持。在性能上面根据[Realm的测试](https://link.jianshu.com?t=https://realm.io/news/introducing-realm/)其高于SQLite两倍多，甩CoreData更不止一个数量级。

## 四：realm用法详细介绍

一：安装

利用cocopods安装

1.把pod "Realm"添加到你的Podfile中。

2.在命令行中执行pod install。

3.将CocoaPods生成的.xcworkspace运用到你的开发项目中即可。

二：创建数据库

1：最简单的方式 使用默认数据库

RLMRealm *realm = [RLMRealm defaultRealm];

2：自定义配置数据库

![img](https:////upload-images.jianshu.io/upload_images/5210585-e3ff8306f23f5090.png?)

下面是创建数据库的config的详细介绍

![img](https:////upload-images.jianshu.io/upload_images/5210585-e05a9eba8793a499.png?)

三：创建表

在realm里创建表其实就是创建一个模型对象，这个模型对象必须继承自Realm Model Object, Realm Model Object顾名思义, 就是Realm 数据库存储的模型对象,



![img](https:////upload-images.jianshu.io/upload_images/5210585-2687c5fb93f8e7b6.png?)



![img](https:////upload-images.jianshu.io/upload_images/5210585-59a2e12877cb69c2.png?)

数据表中一对一(one-to-one)关系来说，只需要声明一个RLMObject子类类型的属性即可，就如上图

通过RLMArray类型的属性您可以定义一个对多关系。RLMArray中可以包含简单类型的RLMObject，其接口与NSMutableArray非常类似。

RLMArray可能会包含多个相同 Realm 对象的引用，即便对象带有主键也是如此。例如，您或许会创建一个空的RLMArray，然后连续三次向其中插入同一个对象；当使用 0、1、2 的索引来访问元素的时候，RLMArray将会返回对应的对象，而所返回的这三个对象都是同一个对象。

RLM_ARRAY_TYPE(Dog) // 定义一个 RLMArray 类型

RLM_ARRAY_TYPE 宏创建了一个协议，从而允许 RLMArray 语法的使用。如果该宏没有放置在模型接口的底部的话，你或许需要提前声明该模型类。如下图所示



![img](https:////upload-images.jianshu.io/upload_images/5210585-d1b307eab068a9a3.png?)

反向关系

链接是单向性的。因此，如果对多关系属性Person.dogs链接了一个Dog实例，而这个实例的对一关系属性Dog.owner又链接到了对应的这个Person实例，那么实际上这些链接仍然是互相独立的。为Person实例的dogs属性添加一个新的Dog实例，并不会将这个Dog实例的owner属性自动设置为该Person。但是由于手动同步双向关系会很容易出错，并且这个操作还非常得复杂、冗余，因此 Realm 提供了“链接对象 (linking objects)”属性来表示这些反向关系。

借助链接对象属性，您可以通过指定的属性来获取所有链接到指定对象的对象。例如，一个Dog对象可以拥有一个名为owners的链接对象属性，这个属性中包含了某些Person对象，而这些Person对象在其dogs属性中包含了这一个确定的Dog对象。您可以将owners属性设置为RLMLinkingObjects类型，然后重写+[RLMObject linkingObjectsProperties]来指明关系，说明ownders中包含了Person模型对象。如上图的RLMLinkingObjects ,重写+[RLMObject linkingObjectsProperties]如下图



![img](https:////upload-images.jianshu.io/upload_images/5210585-5778088590d51aae.png?)

RLMObject常用接口方法

非空字段：requiredProperties

在用SQL描述表格的时候，我们经常会给一些字段加上"NOT NULL"来修饰，表示其必须要有值。这个时候只要实现：+ (nonnull NSArray *)requiredProperties 返回一个成员名的数组就可以了。比如：

\+ (NSArray *)requiredProperties {

return @[@"name"];

}

建立索引：indexedProperties

在使用SQL的时候，为了提高查询效率，通常会为一些字段增减"Index"属性，Realm也是支持这个特性的：   + (nonnull NSArray *)indexedProperties;同样返回一成员名的数据就可以了。比如：

\+ (NSArray *)indexedProperties {

return @[@"name"];

}

默认值：defaultPropertyValues

在使用SQL的时候，对于“NOT NULL”的变量，通常还会给一个默认值，Realm通过一个字典来支持：        + (nullable NSDictionary *)defaultPropertyValues;这里返回的是一个字典，字典的key为一个个属性的名字，value为这个属性的默认值。比如：

\+ (NSDictionary *)defaultPropertyValues {

return @{@"name" : @"Jim", @"age": @12};

}

主键：primaryKey

在二维表中，主键是个至关重要的属性，他表示了那个字段是可以唯一标记一行记录的，不可重复。Realm也是支持这一的特性：

\+ (nullable NSString *)primaryKey;

因为是主键，所以和上面不一样，直接返回主键属性的名字，而不是一个数组。比如：

\+ (NSString *)primaryKey {

return @"id";

}

不用Realm托管：ignoredProperties

因为Realm通过一个结构的成员来表示表中的各个键值，但是如果其中有一部分我们并不想其录入DB怎么办呢？比如在Persion中有个可以由age推导的年龄段（小孩、青年、成年），这个信息因为和age重复不用存储到DB中。为此Realm支持可以不托管RLMObject中的部分成员：

\+ (nullable NSArray *)ignoredProperties;

\+ (NSArray *)ignoredProperties {

return @[@"ageLevel"];

}

这样我们还可以自定义ageLevel的setter和getter，因为不用Realm托管，所以也就需要自己加上strong/weak等修饰了

四：增（realm的增删改查   比较简洁明了，我在这里贴几段代码就可以表示了，详细的可以看api文档）

![img](https:////upload-images.jianshu.io/upload_images/5210585-dcf1eb0c5ae69a66.png?)

realm数据库增加数据

五：删



![img](https:////upload-images.jianshu.io/upload_images/5210585-a620403060be8985.png?)

realm数据库删除数据

六：改

1内容直接更新

![img](https:////upload-images.jianshu.io/upload_images/5210585-83a6983f07f04121.png?)

数据库更新数据

2 根据主键更新

如果您的数据模型中设置了主键的话，那么您可以使用+[RLMObject createOrUpdateInRealm:withValue:]来更新对象，或者当对象不存在时插入新的对象

// 创建一个带有主键的“书籍”对象，作为事先存储的书籍

Book *cheeseBook = [[Book alloc] init];

cheeseBook.title = @"奶酪食谱";

cheeseBook.price = @9000;

cheeseBook.id = @1;

// 通过 id = 1 更新该书籍

[realm beginWriteTransaction];

[Book createOrUpdateInRealm:realm withValue:cheeseBook];

[realm commitWriteTransaction];



七：查

// 查询默认的 Realm 数据库

RLMResults *dogs = [Dog allObjects]; // 从默认的 Realm 数据库中，检索所有狗狗

// 查询指定的 Realm 数据库

RLMRealm *petsRealm = [RLMRealm realmWithPath:@"pets.realm"]; // 获得一个指定的 Realm 数据库

RLMResults *otherDogs = [Dog allObjectsInRealm:petsRealm]; // 从该 Realm 数据库中，检索所有狗狗

**条件查询**

1.使用断言字符串查询:

RLMResults *tanDogs = [Dog objectsWhere:@"color = '棕黄色' AND name BEGINSWITH '大'"];

2.// 使用 NSPredicate 查询

NSPredicate *pred = [NSPredicate predicateWithFormat:@"color = %@ AND name BEGINSWITH %@",

@"棕黄色", @"大"];

RLMResults *tanDogs = [Dog objectsWithPredicate:pred];

3.链式查询

如果我们想获得获得棕黄色狗狗的查询结果，并且在这个查询结果的基础上再获得名字以“大”开头的棕黄色狗狗。

RLMResults *tanDogs = [Dog objectsWhere:@"color = '棕黄色'"];

RLMResults *tanDogsWithBNames = [tanDogs objectsWhere:@"name BEGINSWITH '大'"];

关于查询这块 api文档写的特别详细 详情可以去看api （中文api链接：https://realm.io/cn/docs/objc/latest/#section-3）

八：数据排序

RLMResults 允许您指定一个排序标准，从而可以根据一个或多个属性进行排序。比如说，下列代码将上面例子中返回的狗狗根据名字升序进行排序:

// 排序名字以“大”开头的棕黄色狗狗

RLMResults *sortedDogs = [[Dog objectsWhere:@"color = '棕黄色' AND name BEGINSWITH '大'"]

sortedResultsUsingProperty:@"name" ascending:YES];

## 五 ：通知监听、加密、KVC和KVO

1 RLMObject、RLMResult以及 RLMArray都遵守键值编码(Key-Value Coding)（KVC）机制。当您在运行时才能决定哪个属性需要更新的时候，这个方法是最有用的。将 KVC 应用在集合当中是大量更新对象的极佳方式，这样就可以不用经常遍历集合，为每个项目创建一个访问器了。

RLMResults *persons = [Person allObjects];[[RLMRealm defaultRealm] transactionWithBlock:^{     [[persons firstObject] setValue:@YES forKeyPath:@"isFirst"];// 将每个人的 planet 属性设置为“地球”[persons setValue:@"地球"forKeyPath:@"planet"];}];

2 Realm 对象的大多数属性都遵从 KVO 机制。所有 RLMObject子类的持久化(persisted)存储（未被忽略）的属性都是遵循 KVO 机制的，并且 RLMObject以及 RLMArray中 无效的(invalidated)属性也同样遵循（然而 RLMLinkingObjects属性并不能使用 KVO 进行观察）。

3 realm支持数据库加密

// 产生随机密钥NSMutableData*key = [NSMutableDatadataWithLength:64];

SecRandomCopyBytes(kSecRandomDefault, key.length, (uint8_t *)key.mutableBytes);

// 打开加密文件

RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];

config.encryptionKey = key;

NSError*error =nil;

RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:&error];

if(!realm) 

{

// 如果密钥错误，`error` 会提示数据库不可访问

NSLog(@"Error opening realm: %@", error);

}

Realm 支持在创建 Realm 数据库时采用64位的密钥对数据库文件进行 AES-256+SHA2 加密。这样硬盘上的数据都能都采用AES-256来进行加密和解密，并用 SHA-2 HMAC 来进行验证。每次您要获取一个 Realm 实例时，您都需要提供一次相同的密钥。不过，加密过的 Realm 只会带来很少的额外资源占用（通常最多只会比平常慢10%）

4 通知

// 获取 Realm 通知token = [realm addNotificationBlock:^(NSString*notification, RLMRealm * realm)

 {

 [myViewController updateUI];

}];

[token stop];// 移除通知[realm removeNotification:self.token];

Realm 实例将会在每次写入事务提交后，给其他线程上的 Realm 实例发送通知。一般控制器如果想一直持有这个通知，就需要申请一个属性，strong持有这个通知。

## 六：错误处理

 在数据的处理中可能会出现失败的情况,在查看错误的时候,有相关方法可以使用:

提交事务失败

[realm commitWriteTransaction:(NSError * _Nullable __autoreleasing * _Nullable)]

也可以使用

[realm transactionWithBlock:^{

} error:(NSError * _Nullable __autoreleasing * _Nullable)]

配置数据库失败

RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:&error];

要处理在指定线程中初次 Realm 数据库导致的错误， 给 error 参数提供一个 NSError 指针

## 七： 注意事项

1 跨线程访问数据库，realm一定要新建一个

Terminating app duetouncaught exception'RLMException', reason:'Realm accessed from incorrect thread.

如果出现上条错误那就是因为你访问Realm数据的时候，使用的Realm对象所在的线程和当前线程不一致。

解决办法就是在当前线程重新获取最新的Realm，即可。

2 自己封装一个realm全局单例实例是没啥用的

很多开发者应该都会对Core Data和Sqlite3或者FMDB，自己封装一个类似Helper的单例。于是我也在这里封装了一个单例，在新建完Realm数据库的时候strong持有一个Realm的对象。然后之后的访问中只需要读取这个单例持有的Realm对象就可以拿到数据库了。

想法是好的，但是同一个Realm对象是不支持跨线程操作realm数据库的。

Realm 通过确保每个线程始终拥有 Realm 的一个快照，以便让并发运行变得十分轻松。你可以同时有任意数目的线程访问同一个 Realm 文件，并且由于每个线程都有对应的快照，因此线程之间绝不会产生影响。需要注意的一件事情就是不能让多个线程都持有同一个 Realm 对象的 实例 。如果多个线程需要访问同一个对象，那么它们分别会获取自己所需要的实例（否则在一个线程上发生的更改就会造成其他线程得到不完整或者不一致的数据）。

其实RLMRealm *realm = [RLMRealm defaultRealm]; 这句话就是获取了当前realm对象的一个实例，其实实现就是拿到单例。所以我们每次在子线程里面不要再去读取我们自己封装持有的realm实例了，直接调用系统的这个方法即可，能保证访问不出错。

3 建议每个model 都设置主键，方便add和update

4查询也不能跨线程

Terminating app duetouncaught exception'RLMException', reason:'Realm accessed from incorrect thread

处理方法是在当前线程重新获取最新的realm

5 Realm 没有自动增长属性





作者：子夏的不语
链接：https://www.jianshu.com/p/c483a0de030b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。