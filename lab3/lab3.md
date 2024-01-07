print在各个语言中都是必不可少的一个函数。例如，在C语言中，printf常常被用来输出一些信息或者用来debug，不承担代码逻辑，在C++中，cout承担类似的角色，由于大家没有学过C，下面的例子以C++说明，printf的实际内容为cout。。但是作为一个代码中的隐藏第六人，却会影响代码的执行结果，比如，经常会遇到注释或者增加一个printf，导致程序执行结果错误或者segment fault。这些诡异的bug都是因为对printf在内存模型的影响不够了解，我们这个作业就用一个例子浅探一下printf的世界。


printf对内存模型的影响
printf对内存模型的影响集中在两个方面

打印内容前，会创建一个buffer作为打印内容的缓存，这个buffer的大小由具体的实现而定，是不确定的。助教在x86_64的机器上测试，得出的结果是1024字节。这个buffer的地址是在栈上的，所以会影响栈的大小。
打印内容时，printf函数会对产生一个write syscall，会对printf的buffer进行输出，这个过程会影响到printf的buffer的内容。



背景：在给出的print.cpp中，我们在21行打印了一个字符串，如果我们在程序中注释掉这一行，我们就会出现一个segment fault，这是因为我们在main函数中声明了一个char数组，这个数组的大小是1024，但是我们在main函数中并没有使用这个数组，所以这个数组的地址在栈上是不确定的，但是我们在print函数中使用了这个数组，所以会出现segment fault。
#include <iostream>

using namespace std;

#define array_number 64

int matrix[array_number][array_number];

int **double_array(size_t n) {
    int **result = new int*[8];

        for (int i = 0; i < n; ++i) {
            result[i] = matrix[i];
            for (int j = 0; j < n; ++j){
                result[i][j] = j;
            }
    }

    return result;
}

int main() {
    // cout<<"A magic print! If you comment this, the program will break."<<endl;
    int **result = double_array(array_number);
    
    for (int i = 0; i < array_number; ++i) {
        cout<<"print address of result[i] "<<&result[i][0]<<endl;
        for (int j = 0; j < array_number; j++) {
            result[i][j] = j;
            cout<<"print content of result[i][j] "<<result[i][j]<<endl;
        }
    }
    free(result);
}
需求：
本次实验需要大家在注释掉cout<<"A magic print! If you comment this, the program will break."<<endl;后，修复这个段错误的bug；
然后提交一个文档，具体说明你怎么找到的这个修复方法，并且阐述为什么会出现这个问题，最好能写一段代码从内存布局上基于证明。

