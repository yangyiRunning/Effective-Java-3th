# Effective Java 3th

## 索引

1. [创建和销毁对象](#创建和销毁对象)
2. [对象的能用方法](#对象的能用方法)
3. [类和接口](#类和接口)

---

## 创建和销毁对象

1. **考虑使用静态工厂方法代替构造方法**
    - 优点:
        1. 有名字
        2. 每次调用的时候，不一定要创建新的对象
        3. 可以返回一个类型的子类型 Collections就是这种用法
        4. 返回对象的类可以随调用的不同而变化（用输入的参数值决定返回哪个），如EnumSet
        5. 返回对象的可以不存在，当写这个静态方法时，ServiceLoader
            - ServiceProviderFramework
                - service interface
                - provider registration
                - service access api
                - service provider(可选的，当没有的时候，通过反射获取实现类)
    - 缺点:
        1. 只含有静态工厂方法时，类不能子类化
        2. 现有文档对这种方法支持不好，因此不容易知道怎样去实例化一个对象，解决方法，用现惯用的命名方式

2. **当有多个构造参数时，考虑使用builder模式**

3. **不可实例化的类要有一个private的构造方法**

4. **依赖注入好于硬编码的资源**

5. **避免创建不必要的对象**
    - 用 String str = "abcd"；而不是String str= new String("abcd")
    - 静态工厂方法比构造方法，更能避免创建不必要的对象，对于不可变对象来说

6. **排除过时的对象引用**
    - 当一个类管理自己的内存时，注意内存泄漏，例如：数组，对象引用的数组，而不是数组里面的内容
    - 缓存相关
        1. 当缓存的生命周期由其外部引用决定时用weakreference
        2. 当不是上面这种情况时，起一个线程去扫，删除无用的缓存，或者再加入缓存时，去除不必要的缓存，如removeEldestEntry  LinkedHashmap
        3. listener和callback容易导致内存泄漏，用weakreference

7. **try-with-resources优于try-finally**
    - 用try-with-resources代替try-finally
    - 如果要想使用try-with-recources需要类实现AutoCloseable
    - java7中引入

## 对象的能用方法

1. **当覆写equals时遵守的约定定**
    - 在符合下面情况时，不要覆写equals：对象等于它自己
        1. 对象本质上唯一的
        2. 没有必要提供逻辑上的相等性测试
        3. 基类已经覆写了equals方法，且子类的行为对于继承来的equals的实现是合适的

    - 遵守的约定
        1. 自反性
        2. 对称性
        3. 传递性
            - 没有办法去继承一个类并且添加一个属性，同时保证传递性
            - 用组合代替继承，来解决这个问题
            - 当混用基类和子类时会有问题(当基类是抽象类时没有问题，因为不可以创建抽像类的对象)
        4. 一致性
            - 不要依赖不可信的资源做equals操作
        5. x.equals(null)必返false

    - 实现高质量的equals的方法
        1. 用==检查被比较的对象，与本对象是不是同一个对象
        2. 用 instanceof 检查被比较的对象是否有正确的类型
        3. 转换被比较的对象到正确的类型
        4. 检查重要的域是否相等
        5. 一些注意事项
            - 原始类型除了float 和 double，用==
            - 对象类型用equals
            - Float.compare, Double.compare不要用Float.equlas Double.equals(会导致装箱)
            - 数组
                - 用上面的方法比较重要的元素
                - 如果每个都需要比较的话用Arrays.equlas
            - 有些属性有可能包含空，为了避免空指针用Objects.equlas()方法

    - 其他一些注意事项
        - 当覆写了equals，也要覆写hashCode方法
        - 不要把equals方法中的形参换为其他的，应该为Object
        - 写完后检查是不是满足对称性，传递性，和一致性

2. **当覆写了equals方法时，总是要覆写hashCode方法**
    - 几个约定
        1. 如果两个对象调用equals是相等的，两个对象的hashCode应该返回相同的值
        2. 如果两个对象调用equals方法是不相等的，对于hashCode的返回值，不要求必须不相等，但是不相等的话，可以提高hashtable的性能

    - 写hashCode的方法
        1. 声明int变量result，并将第一个重要的filed算hash赋给它
        2. 如何计算hash值
            - 原始类型，Type.hashCode(f)，type是原始类型对应的装箱类型
            - 引用类型，且该引用类型在equals方法中是调用equals方法进行比较，则调用hashCode算；如果对象是空，则为0；如果是个复杂的比较，先算出该对象的一个标准表示，对这个标准表示计算hash
            - 数组对象，计算里面的每个重要的对象的hash，并用第三步的方法进行组合这些值；如果都很重要用Arrays.hashCode()；如果都不重要，用一个常量值，通常不是0
        3. 组合值
            - result = 31 * result + c(某个对象的hash)
        4. 注意事项
            - 在equals方法中没有用到的对象，在hashCode中也不应该使用
            - Objects.hash方法用在对性能要求不高的地方
            - 当计算hashCode耗时时，可以缓存，但要注意线程安全问题
            - 不要对hashCode做详细的解释，以免客户端对这个值产生特定的逻辑依赖，以至于后面没法改变hashCode的生成算法，以提高性能

3. **总是覆写toString**
    - toString方法应该文档化，并且说明是不是有格式化的，以避免有些用户去依赖这个format
    - 无论toString的返回是有格式的还是无格式，都应该提供对toString包含的属性的get方法，以避免客户去解析字符串
    - 抽象类中，如果子类需要共享一个常见的字符串，应该在这个抽象类中覆写toString方法，如Collection类

4. **谨慎明智的复写clone**
    - 不可变类不要覆写
    - 克隆方法作为另一种构造方法，必须不能破坏原始对象
    - 类可被克隆，属性不能被final修饰
    - clone方法中不能调用子类的的方法，因此可以调用的方法用private或final修饰
    - **不建议覆写**
        - 用拷贝构造
        - 用静态构造
        - final类可以复写为了提供性能
        - 数组可以用克隆
    - 一些技巧
        - 数组被克隆时可以不用作强制类型转换
        - 是线程不安全的，如果要线程安全需要加锁
    - 拷贝的实现的几种方法
        1. 如果只有简单变量的话，简单的调用super.clone即可
        2. 数组：可以调用clone方法，但是里面存储的东西，还是浅拷贝
        3. 有复杂的变量，尤其是数组的情况，对数组里面包含的对象，单独实现深拷贝，然后新构建数组，一个一个复制
        4. 调用super.clone，设置使所有的成员属性都在初始状态，调用高层次的方法重新产生与原始对象一样状态的值
    - 为继承而设计的类
        1. 不实现Clonable接口，但是实现成Object一样，抛出异常，这样子类可以灵活选择，支持还是不支持
        2. 将clone方法写成final完全防止子类支持clone

5. **考虑实现Comparable**
    - 有自然顺序的类，应该实现Comparable
    - 实现规范
        1. sgn(x.compare(y)) == -sgn(y.compare(x))
        2. x.compare(y)抛出异常y.compare(x)也要抛出
        3. 传递性
        4. x.compare(y)==0-->sgn(x.compare(z))==sgn(y.compare(z))
        5. 建议但不要求，(x.compare(y)==0)==(x.equals(y))，如果没有这样实现的话，要在文档中写出
    - 与equals一样，不要在继承关系中实现Comparable，用组合
    - 实现方法
        1. 值类型用Short.compare类似的
        2. 引用类型递归调用compareTo，如果没有实现这个接口的话，用Comparator代替
        3. 有多个重要的属性，按重要性排序比较，有一个不相等的即是比较完成，或者直接用Comparator中的静态构造函数
    - 不要用o1-o2或者o1<o2这种形式，要用Short.compare或者Comparator中的静态构造函数

## 类和接口

1. **使类和成员的可访问性最小**
    - 包访问权限的类或接口，只在包中的一个类中用到，将其实现成使用类的静态私有内部类
    - 同一个包中的类经常需要另一个类中的包访问权限，考虑重新设计类，将这些放到一个类中
    - 成员属性不应该为public
    - 静态成员属性也不应该为public，例外情况是public static final，同时保存原始类型或指向不可变对象
    - 长度非0的数组是可变的，用public static final或者提供返回数组的get方式是错误的
        - 数组为private，返回一个不可变的list，Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES))
        - 数组为private，返回这个数组的clone
    - java9提供的基于包的新的两个访问权限，大多数情况下没必要用

2. **在公有类中使用get方法而不是公有域**
    - 在包外可以访问到的类，提供get方法，而不是公有属性
    - 类是包访问权限或者private nested class，可以直接暴露成员属性（不论是可变的还是不可变的）
    - 公有类中，不可变的属性可以是public的

3. **最小化可变性**
    - 实现方法
        1. 不提供可以修改对象状态的方法
        2. 保证类不能被继承
            - final
            - 将构造的访问性设为private 或 package-private
        3. 使所有属性为private
            - 不可变的属性可以是public的，但是不建议，因为这样在后续版本中不能改变实现了
        4. 所有属性都用final修饰
        5. 保证不能对可变对象有访问权限
            - 不用客户代码提供的可变对象引用，初始这样的属性
            - get方法不能返回这样的属性
            - 在构造函数，get方法，readObject中做防御性拷贝
    - 不可变对象是线程安全的
    - 为不可变的类提供静态构造方法（代替构造方法），以缓存相相同的对象，以获得性能的提高
    - 不需要做防御性拷贝，因此不用提供拷贝构造函数和clone方法
    - 不仅可以共享不可变对象，其内部状态也可以共享，考虑BigInteger的negate方法的实现，不是public的那种共享
    - 不可变对象为其他对象提供了大量的基础构件
    - 不可变对象提供了免费的原子失败机制
    - 缺点
        - 对于不同的值来说，需要不同的对象，会造成性能和内存的损耗
            1. 多步运算可以加大这种缺点，这时可以采用一个包级访问权限的相对应的可变对象来优化，如BigInteger
            2. 提供一个公开的对应的可变对象类如String和StringBuffer
    - BigInteger和BigDecimal是可以被继承的，因此如果写的代码依赖这两个类的不可变性，要做检查，如果不是的话要进行防御性拷贝
    - 可以有一些属性不用final修饰，以用来缓存中间计算结果，以提高性能，内存可见性可能有问题，这样的话可以用volatile去修饰下
    - 一个不可以写成不可变的类，尽可能的限制他的可变性
    - 构造函数创建完全初始化的对象
        - 不在构造器和静态函数之外创建其他的初始化方法
        - 不提供重新初始化方法

4. **组合优于继承**
    - 安全的使用继承
        - 同包中
        - 有专门的文档，且专门是为继承而设计的类，可以继承这个类
    - 用组合代替继承
        - 组合用forwarding包装类的形式实现
    - 包装类不适用于回调框架
    - 使用继承时要确认的问题
        1. B is A，存在真正的类型关系
        2. 被继承的类在api是否有缺陷，如果有缺陷，被继承后的子类将会传播这种缺陷

5. **为了继承去设计和文档化，否则就禁止继承**
    - 文档化
        - 用@implSpec
            - java9(包括)之前没有默认启用
            - -tag "implSpec:a:Implementation Requirements:"
        - 详细写出实现细节
    - 为继承而设计
        - 聪明而审慎的提供内部细节实现的钩子，用protect method，protected fields的方式
        - 测试为继承而设计的类的唯一方法是编写子类，且该类由其他人员编写
        - 一些必须要遵守的规则
            - 构造函数不应该调用被覆写的方法
            - clone、readObject不要调用可覆写的方法
            - 实现Serializable接口时，如果用readResolve和writeReplace方法，要用protected
    - 没能为继承而设计且文档化的类禁止继承
        - 用final
        - 使用构造方法为private或package-private，且加入静态构造方法以代替常规构造方法
    - 一个类必须要被继承时（出于某些原因），完全消除对override方法的调用，并且文档里写清楚

6. **接口优于抽象类**
    - 已经存在类很容易被翻新成实现一个接口的新类
    - 接口对于定义混合类型是理想的
    - 接口可以构建非层级类型的框架
    - 接口通过包裹可以提供安全的，强大的功能增强
    - **接口的一些限制**
        - 不允许提供这样的默认方法，但是可以指定这两个方法的形为，应该实现成啥样，这样也就意味着实现类必须要提供这些方法的实现
            1. equels
            2. object有的方法
            3. hashCode
        - 不能有成员属性
        - 不能有非公开的static方法
    - 编写抽象骨架类的方法
        - 定义出原始方法，需要子类实现的
        - 提供默认实现方法
        - 实现全部剩余的方法（如果前面两个包括了所有方法，就不需要再专门写一个抽象骨架类了，接口自己就满足需求了）
        - other
            - 如果有抽象骨架类的话，可以包括一些非分开的属性和方法
            - 命名方式AbstractInterfaceName

7. **为子孙设计接口**
    - 接口中不是必要的情况下不要用default方法，除了骨架抽象类的实现
    - 接口在release之前，要测试
        - 三种不同的实现类
        - 接口的实现类，去实现不同的任务

8. **仅用接口定义类型**
    - 常量接口模式是对接口的糟糕使用
    - 定义常量的玩法
        - 和类或接口紧密的联系，直接public static final写到相应的类或接口中
        - 如果常量用枚举类型是好的，用枚举，且导出
        - 其他情况用不可实例化的工具类来导出常量
    - 接口仅用于定义类型

9. **类层次优于标签类**
    - 如果类是标签的形式实现的，考虑重构成用类层次实现
    - 标签类只是类层次的苍白的防治品

10. **静态内部类优于非静态的**
    - 静态内部类
        - 静态内部使用场合
            - 是外部类的公开帮助类,public 静态内部类
            - 表示宿主类对象的一个组件，private 静态内部类
        - public 或 protected的静态内部类，会做为导出api的一部分，在后续的版本中不能将其改为非static的内部类
    - 内部类
        - 使用场合: 定义一个适配器，如List中的iterator
    - 匿名内部类
        - 当在静态context的环境中，不会持有外部类的引用，这里就是指被赋值到static成员属性
        - 不能有static members，除了被final修饰的原始类型和String
        - 最好不超过10行，以免对可读性造成伤害
        - 使用场合: 静态工厂方法中返回一个对象
    - 局部类(相当于方法里的局部变量，只能在方法中使用)
        - 当在静态context的环境中，不会持有外部类的引用，这里就是static类
        - 不能有static members，除了被final修饰的原始类型和String
        - 最好不超过10行，以免对可读性造成伤害
        - 局部内部类只能在声明这个内部类的方法中创建对象

11. **在一个源文件中只有一个顶级类的定义**
    - 绝对不要在一个源文件中定义多个顶级类或顶级接口