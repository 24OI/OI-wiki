在默认情况下，cin(cout)是极为迟缓的读入(输出)方式，而scanf(printf)比cin(cout)快得多。

可是为什么会这样呢？有没有什么办法解决读入输出缓慢的问题呢？

## 关闭同步

### sync_with_stdio

这个函数是一个“是否兼容stdio”的开关，C++为了兼容C，保证程序在使用了std::printf和std::cout的时候不发生混乱，将输出流绑到了一起。

这其实是C++为了兼容而采取的保守措施。我们可以在IO之前将stdio解除绑定，这样做了之后要注意不要同时混用cout和printf之类

### tie

tie是将两个stream绑定的函数，空参数的话返回当前的输出流指针。

在默认的情况下cin绑定的是cout，每次执行 &lt;&lt; 操作符的时候都要调用flush，这样会增加IO负担。可以通过tie(0)（0表示NULL）来解除cin与cout的绑定，进一步加快执行效率。

### 代码实现

```cpp
std::ios::sync_with_stdio(false);

std::cin.tie(0);
```

## 读入优化

scanf和printf依然有优化的空间，这就是本章所介绍的内容——读入和输出优化。

-   注意，读入和输出优化均针对整数，不支持其他类型的数据

### 原理

众所周知，getchar()是用来读入char类型，且速度很快，用“读入字符——转换为整形”来代替缓慢的读入

每个整数由两部分组成——符号和数字

整数的’+’通常是省略的，且不会对后面数字所代表的值产生影响，而’-’不可省略，因此要进行判定

10进制整数中是不含空格或除0~9和正负号外的其他字符的，因此在读入不应存在于整数中的字符(通常为空格)时，就可以判定已经读入结束

### 代码实现

```cpp
int read(){

	int x=0,w=1;char ch=0;

    	while(ch<'0' || ch>'9'){//ch不是数字时

		if(ch=='-') w=-1;//判断是否为负

		ch=getchar();//继续读入

	}

	while(ch>='0' && ch<='9') {//ch是数字时

		x=(x<<3)+(x<<1)+ch-'0';//将新读入的数字’加’在x的后面
		//x<<3==x*8  x<<1==x*2  所以(x<<3)+(x<<1)相当于x*10
		//x是int 类型，char类型的ch和’0’会被自动转为其ASCII表中序号，相当于将ch转化为对应数字
		ch=getchar();//继续读入
	}
	return x*w; //数字*正负号==实际数值
}
```

-   举例 

读入num 可写为 num=read();

## 输出优化

### 原理

同样是众所周知，putchar()是输出单个字符，其速度远快于其它输出方式 

因此将数字的每一位转化为字符输出以加速

要注意的是，负号要单独判断输出，并且每次%（mod）取出的是数字末位，因此要倒序输出（递归加速）

### 代码实现

```cpp
int write(int x){

	if (x<0) {//判负+输出负号+变原数为正数
	
    		x=-x;
		
        	putchar('-');
		
    	}
	
	if (x>9) write(x/10);//递归，将除最后一位外的其他部分放到递归中输出
	
	putchar(x%10+'0');//已经输出（递归）完x末位前的所有数字，输出末位
}
```

-   举例

输出num可写为write(num);

也可以写成非递归的形式，来得到更好的效果。

## 参考

<http://www.hankcs.com/program/cpp/cin-tie-with-sync_with_stdio-acceleration-input-and-output.html>

<http://meme.biology.tohoku.ac.jp/students/iwasaki/cxx/speed.html>
