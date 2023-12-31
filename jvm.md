# jvm

## 程序计数器

每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。 

如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是本地（Native）方法，这个计数器值则应为空（Undefined）。此内存区域是唯一一个在《Java虚拟机规范》中没有规定任何OutOfMemoryError情况的区域。 

## Java虚拟机栈

每条线程都需要有一个虚拟机栈,私有.

## 对象的创建

1. 遇到一条字节码new指令时，**首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过**。

2. **如果没有，那必须先执行相应的类加载过程,结束后对象所需内存的大小在类加载完成后便可完全确定**

3. 把**一块确定大小的内存块从Java堆中划分**出来,由Java堆的内存是否规整来决定 采用"指针碰撞"还是"空闲列表的"分配方式分配内存
   

**注意:**Java堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理（Compact）的能力决定

创建对象是一个很频繁的过程,在并发的情况下不是线程安全的,可能出现正在给对象 A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。

解决方案：一种是对分配内存空间的动作进行同步处理——实际上虚拟机是**采用CAS配上失败重试的方式**保证更新操作的原子性；另外一种是把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local AllocationBuffer，TLAB），哪个线程要分配内存，就在哪个线程的本地缓冲区中分配，只有本地缓冲区用完了，分配新的缓存区时才需要同步锁定。虚拟机是否使用TLAB，可以通过-XX：+/-UseTLAB参数来设定。

4. 内存分配完成之后，虚拟机必须将**分配到的内存空间（但不包括对象头）都初始化为零值**，**如果使用了TLAB的话，这一项工作也可以提前至TLAB分配时顺便进行**。这步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，使程序能访问到这些字段的数据类型所对应的零值。 

5. Java虚拟机还要对**对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到 

   类的元数据信息、对象的哈希码（实际上对象的哈希码会延后到真正调用Object::hashCode()方法时才 

   计算）、对象的GC分代年龄等信息。**这些信息存放在对象的对象头**（Object Header）之中。根据虚拟 

   机当前运行状态的不同，如是否启用偏向锁等，**对象头会有不同的设置方式**

6. 虚拟机的视角来看，一个新的对象已经产生了。但是从Java程序的视角看来，对象创建才刚刚开始——构造函数，即Class文件中的<init>()方法还没有执行，所有的字段都为默认的零值，对象需要的其他资源和状态信息也还没有按照预定的意图构造好。一般来说（由字节码流中new指令后面是否跟随invokespecial指令所决定，Java编译器会在遇到new关键字的地方同时生成 

   这两条字节码指令，但如果直接通过其他方式产生的则不一定如此），new指令之后会接着执行<init>()方法，这样一个真正可用的对象才算完全被构造出来


## 对象实例在java堆中的内存布局

总共分为三个部分:**对象头  实例数据  对齐填充**

---

**对象头:分为两个部分**

  		1. **第一部分:** 存储运行时数据  如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等被设计成一个有着它被设计为  动态定义的数据结构，以便在极小的空间内存储尽量多的数据，**根据对象的状态复用自己的存储空间**
                		2. **第二部分:**  类型指针,指向对象的类型元数据,供jvm确定这个实例 来自于哪个对象

**注意: 如果实例是java数组,对象头还要记录这个 数组的长度(因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是数组的长度是不确定的，将无法通过元数据中的信息推断出数组的大小。)**

---

**实例数据:是对象真正存储的有效信息**

记录了在程序代码里面所定义的各种类型的字段内容,包括父类继承下来的,自身自带的

**注意:**这部分的存储顺序会受到虚拟机分配策略参数（-XX：FieldsAllocationStyle参数）和字段在Java源码中定义顺序的影响。

HotSpot虚拟机默认的分配顺序为longs/doubles、ints、shorts/chars、bytes/booleans、oops（OrdinaryObject Pointers，OOPs），从以上默认的分配策略中可以看到，**相同宽度的字段总是被分配到一起存放**，在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。**如果HotSpot虚拟机的+XX：CompactFields参数值为true（默认就为true），那子类之中较窄的变量也允许插入父类变量的空隙之中**，以节省出一点点空间。 

---

**对齐填充:  不是必然存在的**,仅仅起着占位符的作用