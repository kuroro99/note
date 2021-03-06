# runtime理解和linux系统根目录各个文件夹的作用

2022.2.23笔记，主要是对c++中runtime多态的理解以及对linux根目录各个文件夹作用的解释。

## c++中runtime重载和compiletime重载

核心区别在编译过程中，对于使用了虚函数的继承使用的是runtime重载，即在父类和继承的子类中加入vptr指针，指向对应的虚函数。在编译过程中只需对类中的vptr指针的调用进行编译，而不需要对函数体进行编译，在运行的过程中从vptr指向的虚函数表来寻找需要的重载函数。这样的优点在于减少编译压力，但是会浪费一部分资源。

而正常的重载（即在compiletime进行重载）在调用子类重载函数时需要将先找到函数体，将函数体进行编译执行，这样在频繁调用重载函数时会加大编译的压力。

举个例子：

``````c++
class Parent{
    public:
        Parent(int a=0){
            this->a = a;}
        virtual void print(){ }  
    private:
        int a;
};

class Son:public Parent{
    public:
       Son(int a=0,int b=0):Parent(a){
           this->b = b;}
       void print(){
           cout<<"Son"<<endl;}
    private:
        int b;
    };

  void play(Parent *p){ 
        p->print();}

    void main(int argc, char const *argv[])
    {
        Parent p; 
        Son s;
        play(&s);
        return 0;
    }
``````

这是一个虚函数即runtime实现的函数重载，在调用`play(&s)`这个函数的过程中，编译完成后只需要让计算机知道这个函数中只需要调用输入参数的vptr指针找到对应的print()实现即可。

再看一个compiletime的例子

``````c++
class Parent{
    public:
        Parent(int a=0){
            this->a = a;}
        void print(){ 
            cout<<"parent"<<endl;}  
    private:
        int a;
};

class Son:public Parent{
    public:
       Son(int a=0,int b=0):Parent(a){
           this->b = b;}
       void print(){
           cout<<"Son"<<endl;}
    private:
        int b;
    };

  void play(Parent *p){ 
        p->print();}

    void main(int argc, char const *argv[])
    {
        Parent p; 
        Son s;
        play(&s);
        return 0;
    }
``````

这里是对父类直接进行的重载，此时对于调用`play(&s)`的时候，编译器需要让计算机知道这个print()函数是使用的哪个被重载的函数，故会找到对应的son实现的print()函数在编译过程中将对应的内容编译在调用`play(&s)`的过程中。

## linux根目录各个文件的作用

注意：usr-->lib   默认存放的动态库，自己写的应用程序/home/app里面的文件都会调用此
/usr-->lib 目录里面的动态库。
以下是linux系统常见的重要目录以及各个目作用：
/
 根目录。
包含了几乎所的文件目录。相当于中央系统。进入的最简单方法是：cd /。

/boot
引导程序，内核等存放的目录。
这个目录，包括了在引导过程中所必需的文件，引导程序的相关文件（例如grub，lilo以及相应的配置文件以及Linux操作系统内核相关文件（例如vmlinuz等一般都存放在这里。在最开始的启动阶段，通过引导程序将内核加载到内存，完成内核的启动（这个时候，虚拟文件系统还不存在，加载的内核虽然是从硬盘读取的，但是没经过Linux的虚拟文件系统，这是比较底层的东西来实现的。然后内核自己创建好虚拟文件系统，并且从虚拟文件系统的其他子目录中（例如/sbin 和 /etc加载需要在开机启动的其他程序或者服务或者特定的动作（部分可以由用户自己在相应的目录中修改相应的文件来配制。如果我们的机器中包含多个操作系统，那么可以通过修改这个目录中的某个配置文件（例如grub.conf来调整启动的默认操作系统，系统启动的择菜单，以及启动延迟等参数。

/sbin
超级用户可以使用的命令的存放目录。
存放大多涉及系统管理的命令（例如引导系统的init程序，是超级权限用户root的可执行命令存放地，普通用户无权限执行这个目录下的命令（但是时普通用户也可能会用到。这个目录和/usr/sbin; /usr/X11R6/sbin或/usr/local/sbin等目录是相似的，我们要记住，凡是目录sbin中包含的都是root权限才能执行的，这样就行了。后面会具体区分。

/bin
普通用户可以使用的命令的存放目录。
系统所需要的那些命令位于此目录，比如ls、cp、mkdir等命令；类似的目录还/usr/bin，/usr/local/bin等等。这个目录中的文件都是可执行的、普通用户都可以使用的命令。作为基础系统所需要的最基础的命令就是放在这里。

/lib
根目录下的所程序的共享库目录。
此目录下包含系统引导和在根用户执行命令时候所必需用到的共享库。做个不太好但是比较形象的比喻，点类似于Windows上面的system32目录。理说，这里存放的文件应该是/bin目录下程序所需要的库文件的存放地，也不排除一些例外的情况。类似的目录还/usr/lib，/usr/local/lib等等。

/dev
设备文件目录。
在Linux中设备都是以文件形式出现，这里的设备可以是硬盘，键盘，鼠标，网卡，终端，等设备，通过访问这些文件可以访问到相应的设备。设备文件可以使用mknod命令来创建，具体参见相应的命令；而为了将对这些设备文件的访问转化为对设备的访问，需要向相应的设备提供设备驱动模块（一般将设备驱动编译之后，生成的结果是一个*.ko类型的二进制文件，在内核启动之后，再通过insmod等命令加载相应的设备驱动之后，我们就可以通过设备文件来访问设备了。一般来说，想要Linux系统支持某个设备，只要个东西：相应的硬件设备，支持硬件的驱动模块，以及相应的设备文件。

/home
普通用户的家目录（$HOME目录。
在Linux机器上，用户主目录通常直接或间接地置在此目录下。其结构通常由本地机的管理员来决定。通常而言，系统的每个用户都自己的家目录，目录以用户名作为名字存放在/home下面（例如quietheart用户，其家目录的名字为/home/quietheart。该目录中保存了绝大多数的用户文件(用户自己的配置文件，定制文件，文档，数据等)，root用户除外（参见后面的/root目录。由于这个目录包含了用户实际的数据，通常系统管理员为这个目录单独挂载一个独立的磁盘分区，这样这个目录的文件系统格式就可能和其他目录不一样了（尽管表面上看，这个目录还是属于根目录的一棵子树上），有利于数据的维护。

/root
用户root的$HOME目录
系统管理员(就是root用户或超级用户)的主目录比较特殊，不存放在/home中，而是直接放在/root目录下了。

/etc
全局的配置文件存放目录。
系统和程序一般都可以通过修改相应的配置文件，来进行配置。例如，要配置系统开机的时候启动那些程序，配置某个程序启动的时候显示什么样的风格等等。通常这些配置文件都集中存放在/etc目录中，所以想要配置什么东西的话，可以在/etc下面寻找我们可能需要修改的文件。一些大型套件，如X11，在 /etc 下它们自己的子目录。系统配置文件可以放在这里或在 /usr/etc。 不过所程序总是在 /etc 目录下查找所需的配置文件，你也可以将这些文件链接到目录 /usr/etc。另外，还一个需要注意的常见现象就是，当某个程序在某个用户下运行的时候，可能会在该用户的家目录中生成一个配置文件（一般这个文件最开始就是/etc下相应配置文件的拷贝，存放相应于“当前用户”的配置，这样当前用户可以通过配置这个家目录的配置文件，来改变程序的行为，并且这个行为只是该用户特的。原因就是：一般来说一个程序启动，如果需要读取一些配置文件的话，它会首先读取当前用户家目录的配置文件，如果存在就使用；如果不存在它就到/etc下读取全局的配置文件进而启动程序。就是这个配置文件不自动生成，我们手动在自己的家目录中创建一个文件的话，也有许多程序会首先读取到这个家目录的文件并且以它的配置作为启动的选项（例如我们可以在家目录中创建vim程序的配置文件.vimrc，来配置自己的vim程序。

/usr
这个目录中包含了命令库文件和在通常操作中不会修改的文件。
这个目录对于系统来说也是一个非常重要的目录，其地位类似Windows上面的”Program Files”目录（请原谅我可能这样做比较不太恰当^_^。安装程序的时候，默认就是安装在此文件内部某个子文件夹内。输入命令后系统默认执行/usr/bin下的程序（当然，前提是这个目录的路径已经被添加到了系统的环境变量中。此目录通常也会挂载一个独立的磁盘分区，它应保存共享只读类文件，这样它可以被运行Linux的不同主机挂载。

/usr/lib
目标库文件，包括动态连接库加上一些通常不是直接调用的可执行文件的存放位置。
这个目录功能类似/lib目录，理说，这里存放的文件应该是/bin目录下程序所需要的库文件的存放地，也不排除一些例外的情况。

/usr/bin
一般使用者使用并且不是系统自检等所必需可执行文件的目录。
此目录相当于根文件系统下的对应目录（/bin，非启动系统，非修复系统以及非本地安装的程序一般都放在此目录下。

/usr/sbin
管理员使用的非系统必须的可执行文件存放目录。
此目录相当于根文件系统下的对应目录（/sbin，保存系统管理程序的二进制文件，并且这些文件不是系统启动或文件系统挂载 /usr 目录或修复系统所必需的。

/usr/share
存放共享文件的目录。
在此目录下不同的子目录中保存了同一个操作系统在不同构架下工作时特定应用程序的共享数据(例如程序文档信息)。使用者可以找到通常放在 /usr/doc 或 /usr/lib 或 /usr/man 目录下的这些类似数据。

/usr/include
C程序语言编译使用的头文件。
linux下开发和编译应用程序所需要的头文件一般都存放在这里，通过头文件来使用某些库函数。默认来说这个路径被添加到了环境变量中，这样编译开发程序的时候编译器会自动搜索这个路径，从中找到你的程序中可能包含的头文件。

/usr/local
安装本地程序的一般默认路径。
当我们下载一个程序源代码，编译并且安装的时候，如果不特别指定安装的程序路径，那么默认会将程序相关的文件安装到这个目录的对应目录下。例如，安装的程序可执行文件被安装（安装实质就是复制到了/usr/local/bin下面，此程序（可执行文件所需要依赖的库文件被安装到了/usr/local/lib目录下，被安装的软件如果是某个开发库（例如Qt，Gtk等那么相应的头文件可能就被安装到了/usr/local/include中等等。也就是说，这个目录存放的内容，一般都是我们后来自己安装的软件的默认路径，如果择了这个默认路径作为软件的安装路径，被安装的软件的所文件都限制在这个目录中，其中的子目录就相应于根目录的子目录。

/proc
特殊文件目录。
这个目录采用一种特殊的文件系统格式（proc格式，内核支持这种格式。其中包含了全部虚拟文件。它们并不保存在磁盘中，也不占据磁盘空间(尽管命令ls -c会显示它们的大小)。当您查看它们时，您实际上看到的是内存里的信息，这些文件助于我们了解系统内部信息。例如：
├1/ 关于进程1的信息目录。每个进程在/proc 下一个名为其进程号的目录。
├cpuinfo 处理器信息，如类型、制造商、型号和性能。
├devices 当前运行的核心配置的设备驱动的列表。
├dma 显示当前使用的DMA通道。
├filesystems 核心配置的文件系统。
├interrupts 显示使用的中断，and how many of each there have been.
├ioports 当前使用的I/O端口。
├kcore 系统物理内存映象。与物理内存大小一样，但实际不占这么多内存；
├kmsg 核心输出的消息。也被送到syslog 。
├ksyms 核心符号表。
├loadavg 系统”平均负载”；3个没意义的指示器指出系统当前的工作量。
├meminfo 存储器使用信息，包括物理内存和swap。
├modules 当前加载了哪些核心模块。
├net 网络协议状态信息。
├self 到查看/proc 的程序的进程目录的符号连接。
├stat 系统的不同状态
├uptime 系统启动的时间长度。
└version 核心版本。

/opt
可择的文件目录。
这个目录表示的是可择的意思，些自定义软件包或者第方工具，就可以安装在这里。比如在Fedora Core 5.0中，OpenOffice就是安装在这里。些我们自己编译的软件包，就可以安装在这个目录中；通过源码包安装的软件，可以把它们的安装路径设置成/opt这样来安装。这个目录的作用一点类似/usr/local。

/mnt
临时挂载目录。
这个目录一般是用于存放挂载储存设备的挂载目录的，比如磁盘，光驱，网络文件系统等，当我们需要挂载某个磁盘设备的时候，可以把磁盘设备挂载到这个目录上去，这样我们可以直接通过访问这个目录来访问那个磁盘了。一般来说，我们最好在/mnt目录下面多建立几个子目录，挂载的时候挂载到这些子目录上面，因为通常我们可能不仅仅是挂载一个设备吧?

/media
挂载的媒体设备目录。
挂载的媒体设备目录，一般外部设备挂载到这里，例如cdrom等。比如我们插入一个U盘，我们一般会发现，Linux自动在这个目录下建立一个disk目录，然后把U盘挂载到这个disk目录上，通过访问这个disk来访问U盘。

/var
内容经常变化的目录。
此目录下文件的大小可能会改变，如缓冲文件，日志文件，缓存文件，等一般都存放在这里。

/tmp
临时文件目录。
该目录存放系统中的一些临时文件，文件可能会被系统自动清空。的系统直接把tmpfs类型的文件系统挂载到这个目录上，tmpfs文件系统由Linux内核支持，在这个文件系统中的数据，实际上是内存中的，由于内存的数据断电易失，当系统重新启动的时候我们就会发现这个目录被清空了。

/lost+found 
恢复文件存放的位置。
当系统崩溃的时候，在系统修复过程中需要恢复的文件，可能就会在这里被找到了，这个目录一般为空。

另外，有些目录初学者容易混淆，这里简单区分一下：
/bin,/sbin与/usr/bin,/usr/sbin:
/bin一般存放对于用户和系统来说“必须”的程序（二进制文件）。
/sbin一般存放用于系统管理的“必需”的程序（二进制文件），一般普通用户不会使用，根用户使用。
/usr/bin一般存放的只是对用户和系统来说“不是必需的”程序（二进制文件）。
/usr/sbin一般存放用于系统管理的系统管理的不是必需的程序（二进制文件）。

/lib与/usr/lib:
/lib和/usr/lib的区别类似/bin,/sbin与/usr/bin,/usr/sbin。
/lib一般存放对于用户和系统来说“必须”的库（二进制文件）。
/usr/lib一般存放的只是对用户和系统来说“不是必需的”库（二进制文件）。
————————————————
版权声明：本文为CSDN博主「sweetfather」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/sweetfather/article/details/79625482

