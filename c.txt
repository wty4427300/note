gcc编译的过程预处理，编译，汇编，连接。
多线程私有数据pthread_key_create。多线程私有数据，特点是一键多值，键可以被所有线程访问，不同线程根据键获取不同的值。他由两个参数第一个参数是指向键值的指针，第二个参数指明了destructor函数，如果这个参数不为空。那么当每个线程结束时，系统将调用这个函数来释放绑定在这个键上的内存块。其次键的内存释放，并不会释放相对应的值的内存，所以应该先释放线程数据在释放键的内存。

int pthread_setspecific(pthread_key_t key,const void *pointer);

pthread_self()//获取线程自身的id；

动态链接库：在程序编译时并不会被连接到目标代码中，而是在程序运行时才被载入，因此在程序运行时还需要动态库存在。

静态连接库：在程序编译时会被连接到目标代码中，程序运行时将不再需要该静态代码。

一个简单的主目录cmake
cmake_minimum_required(VERSION 3.14)
project(untitled7 C)
include_directories()


LINK_DIRECTORIES(lib)
LINK_LIBRARIES(-lpthread -lm)
#find_package (Threads)
set(CMAKE_C_STANDARD 99)

add_executable(untitled7 main.c)
#设置动态链接库目录
TARGET_LINK_LIBRARIES(untitled7 libzlog.so)
TARGET_LINK_LIBRARIES(untitled7 libzlog.so.1)
TARGET_LINK_LIBRARIES(untitled7 libzlog.so.1.2)
#TARGET_LINK_LIBRARIES(untitled7 ${CMAKE_THREAD_LIBS_INIT})

什么是回调函数，即函数作为参数，写法就是参数表里由函数指针，使用的时候把函数传进去就行。

//域位说白了就是用二进制来组织结构体，一个字节（8位）是一个位域，一个位域存不下，会从下一个位域开始存。域位可以是无名域位，无名域位是不能使用的，只是用来做填充或者调整位置的。
struct a{
    int a:8;
    int d:6;
    int  :4;
    int b:2;
    int c:6;
}data;

共用体，允许你在相同内存位置存储不同的数据类型。共用体的内存足以存储共用体中最大的成员变量在此共用体中最大的成员变量是数组所以Data将占用20字节的内存空间。
union Data
{
   int i;
   float f;
   char  str[20];
};
这是一个结构体他的一个变量是Book1
struct Books1
{
    char  title[50];
    char  author[50];
    char  subject[100];
    int   book_id;
} Book1;
这里使用了typedef
typedef struct Books,也就是给struct Books起了个新名字Book
{
    char  title[50];
    char  author[50];
    char  subject[100];
    int   book_id;
} Book;

extern存储类用于一个全局变量的引用，全局变量对所有的程序文件都是可见的，当您使用extern时，对于无法初始化的变量，会把变量名指向一个之前定义过的存储位置。当你由多个文件且定义了一个可以在其他文件中使用的全局变量或函数时们可以在其他文件中使用extern来得到已定义的变量或者函数的引用，可以这么理解，extern是用来在另一个文件中声明全局变量或函数。说白就是可以让变量和函数跨文件。

可以使用stderr文件流来输出错误。一般配合fprintf(stderr,"")这个函数一起使用。

apr库的使用，用base64做简单的介绍
假如我们要apr_base64_encode(),首先就要使用apr_base64_encode_len (int len)，然后使用apr_base64_encode(char *coded_dst, const char *plain_src, int len_plain_src),三个参数分别为，保存base64的结果，需要base64的的字符串，已经该串的长度。相应的decode的过程也是这样的。

c内存管理






