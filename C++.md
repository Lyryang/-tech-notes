# 设计哲学

![image-20241105001946139](./Typora图片/image-20241105001946139.png)

# 环境

## 编程步骤

编写代码：需要代码编译器

将代码变成程序：需要编译器

运行程序：需要操作系统

代码编译器选择vscode

编译器选择GCC

## 代码编译器vscode

跟上国际标准，代码缩进改成2

![image-20241014203513514](./Typora图片/image-20241014203513514.png)

vscode底部也能调整缩进

![image-20241014203522486](./Typora图片/image-20241014203522486.png)

c++代码文件后缀为cpp

![image-20241014203533181](./Typora图片/image-20241014203533181.png)

## VSCode 插件：

Chinese: 中文包
C/C++: c++ 智能提示
FiraCode font: 字体
One Monokai Theme: 主题配色
vscode-luogu: 洛谷平台

<img src="./Typora图片/image-20241014203633305.png" alt="image-20241014203633305" style="zoom:80%;" />

## 编译器

windows 安装 gcc

MSYS2 官网 下载 msys2

[https://www.msys2.org/]

> 软件安装到 50% 会卡主，不用管，十分钟后会自动结束

进度到这个程度

![image-20241014205423577](./Typora图片/image-20241014205423577.png)

打开

![image-20241014205428890](./Typora图片/image-20241014205428890.png)

粘贴，回车确定，在输入y确定

pacman -S mingw-w64-ucrt-x86_64-gcc

`  pacman -S mingw-w64-ucrt-x86_64-gcc  `

![image-20241014205651836](./Typora图片/image-20241014205651836.png)

## 修改环境变量:

**告诉系统执行编译的时候去哪里找编译器**

1. 按一下 win 或点击开始按钮
2. 输入 env，搜索「编辑系统环境变量」
3. 点击「环境变量」
4. 找到「path」，双击或选中后点编辑按钮
5. 点击「新建」按钮，粘贴 c++ 编译器的安装目录

> 注意，是 c++ 编译器所在文件夹的路径，而不是 c++ 编译器的路径，类似这个：
> C:\msys64\ucrt64\bin

在安装编译器的文件夹中找到该文件，并复制路径

<img src="./Typora图片/image-20241014205942786.png" alt="image-20241014205942786" style="zoom: 67%;" />

<img src="./Typora图片/image-20241014210045232.png" alt="image-20241014210045232" style="zoom:67%;" />

<img src="./Typora图片/image-20241014210131511.png" alt="image-20241014210131511" style="zoom: 67%;" />

<img src="./Typora图片/image-20241014210221844.png" alt="image-20241014210221844" style="zoom: 67%;" />

将刚才找到的路径复制并粘贴进去

<img src="./Typora图片/image-20241014210327858.png" alt="image-20241014210327858" style="zoom: 67%;" />

## 编译指令

<img src="./Typora图片/image-20241014210935773.png" alt="image-20241014210935773" style="zoom:67%;" />

详细版
`  c++ test.cpp -o test -O2 --std=c++20 -Wall  `
简单版(简单版会默认生成的文件名为 a.exe 或 a.out 的可执行程序)
`  c++ test.cpp  `

> 如果错误提示： Fatal error: can't create C:\Users\
> 大概率原因是你电脑的用户名是中文导致的
> 这时候就需要在【环境变量】(和修改 PATH 的步骤类似)中找到 TEMP 和 TMP，将它修改为没有中文目录的路径
> 上方【xxx的用户变量】和下方【系统变量】都需要修改
> 可以在 C 盘中新建一个 temp 文件夹，然后将 TEMP 和 TMP 设置为 C:\temp

> 修改完如果又提示了“无法将c++项识别为 cmdlet”
> 可能需要重新修改之前配置的 path，可以删除后再添加

## 运行指令

`  ./test  `
或
`  ./test.exe  `

<img src="./Typora图片/image-20241014214625123.png" alt="image-20241014214625123" style="zoom:67%;" />

## 配置快捷编译运行脚本

shell是操作系统和用户的接口，让用户直接指示操作系统执行命令

<img src="./Typora图片/image-20241015201434448.png" alt="image-20241015201434448" style="zoom: 33%;" />

将下面这个脚本文件复制到一个文件夹中

cr.ps1，ps1是Powershell脚本文件的扩展名

```powershell
param(
  [string]$fileName,
  [string]$inputFileName
)
# 切换为中文
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

function Write-Log {
  param (
    [string]$Message,
    [System.ConsoleColor]$Color = [System.ConsoleColor]::Red
  )
  Write-Host "===== $Message =====" -ForegroundColor $Color
}

# 检查 cr 指令后是否跟着文件名
if (-not $fileName) {
  Write-Log "请在 cr 指令后跟上文件名，如 test.cpp"
  exit 1
}

# 删除 .cpp 后缀
if ($fileName.EndsWith(".cpp")) {
  $fileName = $filename -replace "\.cpp$", ""
}

# 检查cpp文件是否存在
if (-not (Test-Path "$fileName.cpp")) {
  Write-Log "未找到 $fileName.cpp，请查看 VSCode 左侧的资源管理器是否有该程序"
  exit 1
}

$gccOutput = c++ "$fileName.cpp" -o $fileName -O2 --std=c++20 -Wall `
             -fdiagnostics-color=always 2>&1
$exitCode = $LASTEXITCODE
$gccOutput | ForEach-Object { Write-Host $_ }
if ($exitCode -ne 0) {
  if ($gccOutput -match "C:\\Users") {
    Write-Log "用户名不能为中文，请根据说明文件指引进行修改"
    exit 1
  }
  if ($gccOutput -match "WinMain") {
    Write-Log "程序中没有 main 函数，请检查是否保存"
    exit 1
  }
  Write-Log "编译失败，请检查上述语法错误信息后进行修改"
  exit 1
}

if ($gccOutput -match "warning:") {
  Write-Log "上述是警告信息，请检查（如在预期内则可忽略）" Yellow
}
Write-Log "编译成功，开始执行(按 CTRL+c 可强制终止正在执行的程序)" Green

if ($inputFileName) {
  if (-not(Test-Path $inputFileName)) {
    Write-Log "未找到输入文件 $inputFileName，请查看 VSCode 左侧的资源管理器是否有该文件"
    exit 1
  }
  Get-Content $inputFileName | & "./$fileName.exe"
} else {
  & "./$fileName.exe"
}
Write-Log "执行结束，删除可执行程序" Green

Remove-Item "./$fileName.exe"
```

复制cr.ps1文件路径到系统环境变量中

![image-20241015202122964](./Typora图片/image-20241015202122964.png)

重启vscode，使用快捷方式运行脚本，报错

![image-20241015202333334](./Typora图片/image-20241015202333334.png)

windows默认不允许自定义脚本运行

以管理员身份运行powershell

![image-20241015202435997](./Typora图片/image-20241015202435997.png)

输入命令 Set-ExecutionPolicy RemoteSigned 可以解除限制

![image-20241015202604706](./Typora图片/image-20241015202604706.png)

## 快速编译指令

然后输入cr test，成功快捷编译

![image-20241015202859035](./Typora图片/image-20241015202859035.png)

![image-20241015202957388](./Typora图片/image-20241015202957388.png)

## 练习平台

www.luogu.com.cn

![image-20241015205019013](./Typora图片/image-20241015205019013.png)

反馈

![image-20241015205122900](./Typora图片/image-20241015205122900.png)

可在vscode中安装插件

![image-20241015205246703](./Typora图片/image-20241015205246703.png)

对插件进行设置

![image-20241015205421821](./Typora图片/image-20241015205421821.png)

![image-20241015205437205](./Typora图片/image-20241015205437205.png)

setting.json中设置

![image-20241015205510129](./Typora图片/image-20241015205510129.png)

设置快捷键

![image-20241015205849471](./Typora图片/image-20241015205849471.png)

查看题目

![image-20241015205632891](./Typora图片/image-20241015205632891.png)

提交代码

![image-20241015205651193](./Typora图片/image-20241015205651193.png)

根据文件名按快捷键直接对应到题目

![image-20241015205746577](./Typora图片/image-20241015205746577.png)

# 基础概念

## c语言基本概念

计算机理解c++时，会先找main函数去执行

#include	引入其他 c++ 文件
stdio.h	标准输入输出
main	程序的入口函数
indent	缩进，书写格式
printf	格式化输出函数字符串
"Hello World!\n"	双引号包含转义字符，换行语句分隔符

```
#include "stdio.h"
int main() {
  printf("Hello World!\n");
}
```

![image-20241018004512746](./Typora图片/image-20241018004512746.png)

关于头文件

![image-20241018004937623](./Typora图片/image-20241018004937623.png)

 转义字符

![image-20241018005213406](./Typora图片/image-20241018005213406.png)

分号代表这句代码结束了，相当于断句用的符号，如果没有分号，程序就没法理解你的意图

![image-20241018005250377](./Typora图片/image-20241018005250377.png)

## C++基本概念

iostream	标准输入输出库
namespace	命名空间，解决重名冲突
using namespace	本文件使用命名空间
std	标准库的命名空间
cout	标准输出流对象

<<	输出流操作符

endl	换行

```
#include <iostream>
using namespace std;
int main() {
  // 定义一个名字为 a，类型为 int，初始值为 5 的变量
  int a = 5;
  cout << a << endl; // 输出名字为 a 的变量

  /*
   * 定义一个名字为 b
   * 类型为 int
   * 初始值为 5 的变量
   */
  int b = 5;
  cout << b /* 输出名字为 b 的变量 */ << endl;

}
```

![image-20241019170553832](./Typora图片/image-20241019170553832.png)

![image-20241019170816761](./Typora图片/image-20241019170816761.png)

![image-20241019171015984](./Typora图片/image-20241019171015984.png)

![image-20241019171126798](./Typora图片/image-20241019171126798.png)

## 变量

程序的本质是处理数据
数据的本质是 0 和 1
程序中通过变量来表示数据
变量的定义由数据类型、变量名、初始化组成数据类型 int 由 32 个 01 组成，范围 -2^31 ~ 2^31-1

变量名只能使用大小写字母，数字，下划线，其中数字不能做开头

初始化可以使变量等于对应的值，也可缺省

可以通过逗号分隔来便捷的定义多个变量

### 定义变量

![image-20241020214142322](./Typora图片/image-20241020214142322.png)

### 比特

![image-20241020214535187](./Typora图片/image-20241020214535187.png)

### 字节

![image-20241020214551757](./Typora图片/image-20241020214551757.png)

每个字节有固定地址，用16进制数表示，所以说可以通过找到对应的地址改变数据，指向这个变量地址的东西就叫做标识符或者说是变量名

![image-20241020214632584](./Typora图片/image-20241020214632584.png)

变量名

![image-20241020214910196](./Typora图片/image-20241020214910196.png)

### 定义多个变量

![image-20241020215037007](./Typora图片/image-20241020215037007.png)

## 注释

![image-20241020215423985](./Typora图片/image-20241020215423985.png)

![image-20241020215443194](./Typora图片/image-20241020215443194.png)

## 输入与输出

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  cout << b << endl;
}
```



### cin

作用是将终端中的字符给到a这个变量中，把输入的字符转换成a所对应的int类型变量(或者说是数据格式)

![image-20241021225927486](./Typora图片/image-20241021225927486.png)

![image-20241021230130450](./Typora图片/image-20241021230130450.png)

### C与C++输入输出格式对比

单个

![image-20241021230452943](./Typora图片/image-20241021230452943.png)

多个

![image-20241021230528686](./Typora图片/image-20241021230528686.png)

### 输出特殊格式对比

![image-20241021230617441](./Typora图片/image-20241021230617441.png)

### format

该函数根据参数返回一个字符串

![image-20241021230836900](./Typora图片/image-20241021230836900.png)

更改对应设置防止报错，ctrl+，打开设置

![image-20241021231048555](./Typora图片/image-20241021231048555.png)

# 数据类型与运算

## 基础数据类型

可以通过sizeof语句确定一个类型占用多少个字节

![image-20241024204731202](./Typora图片/image-20241024204731202.png)

### 整形

不同数据类型所占字节

![image-20241024204805986](./Typora图片/image-20241024204805986.png)

无符号整数，符号位为正数，比普通整数多了一半的范围

![image-20241024205206361](./Typora图片/image-20241024205206361.png)

![image-20241024205335916](./Typora图片/image-20241024205335916.png)

无符号型加上unsigned

![image-20241025003025469](./Typora图片/image-20241025003025469.png)

### 字符型

初始化

![image-20241025003546694](./Typora图片/image-20241025003546694.png)

![image-20241025003414325](./Typora图片/image-20241025003414325.png)

char类型在输出时，c++会将数字和字符进行一一映射，

映射规则参照ascll码

![image-20241025003719017](./Typora图片/image-20241025003719017.png)

字符变量赋值97后输出

![image-20241025003821780](./Typora图片/image-20241025003821780.png)

### 布尔型

初始化

![image-20241025004323793](./Typora图片/image-20241025004323793.png)

内存中的表示方法，计算机处理的基本单位都是1个字节，8个比特，所以布尔变量也是用一个字节表示

![image-20241025004412382](./Typora图片/image-20241025004412382.png)

### 浮点型

浮点型的内存表示是将一个十进制小数转换为二进制的科学计数法，整数部分固定为1

![image-20241025004821363](./Typora图片/image-20241025004821363.png)

![image-20241025005046582](./Typora图片/image-20241025005046582.png)

10进制科学计数法类比

![image-20241105000504094](./Typora图片/image-20241105000504094.png)

`3.14` 的双精度浮点表示为：

```
0 10000000000 1001001000011111101101010100010001000010110100011000
```

由于这种表示法下，小数点由指数和尾数的值确定，因此小数点不确定，由此叫浮点数

原因：

1. **有限精度**：浮点数的存储位数有限，因此尾数只能近似表示。
2. **无法精确表示某些十进制数**：许多简单的十进制小数在二进制中是无限循环小数，只能近似表示。

#### 初始化

![image-20241025005836656](./Typora图片/image-20241025005836656.png)

![image-20241025010001966](./Typora图片/image-20241025010001966.png)

### auto

自动识别类型，比如下面代码输出

![image-20241025010108705](./Typora图片/image-20241025010108705.png)

## 不同数据类型输出

如何输出不同的数据类型

### 整形输出

#### cout

将所有整形以8进制输出，输出前要设置格式

![image-20241025215953798](./Typora图片/image-20241025215953798.png)

![image-20241025220300788](./Typora图片/image-20241025220300788.png)

#### format

十进制，八进制，十六进制，大写十六进制，带进制符号输出，二进制

![image-20241025220615089](./Typora图片/image-20241025220615089.png)

### 字符型输出

printf以字符c表示，cout和format无需声明

![image-20241025221024181](./Typora图片/image-20241025221024181.png)

### 布尔型输出

![image-20241025221620296](./Typora图片/image-20241025221620296.png)

### 浮点型输出

#### printf

![image-20241025222949701](./Typora图片/image-20241025222949701.png)

#### cout

![image-20241025223240624](./Typora图片/image-20241025223240624.png)

#### format

![image-20241025223413687](./Typora图片/image-20241025223413687.png)

### 设置输出宽度

宽度为7，左右对齐

![image-20241025223811976](./Typora图片/image-20241025223811976.png)

### format占位符{}

冒号前指定输出那个数据，冒号后指定输出格式

![image-20241025223923485](./Typora图片/image-20241025223923485.png)

### 总结

![image-20241025224036758](./Typora图片/image-20241025224036758.png)

## 实例

### 输出宽度

![image-20241026204310676](./Typora图片/image-20241026204310676.png)

```c++
#include "stdio.h"
int main() {
  int a, b, c;
  scanf("%d%d%d", &a, &b, &c);
  printf("%8d %8d %8d\n", a, b, c);
}
```

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  cout.width(8); //设置输出宽度为8
  cout << a << " ";
  cout.width(8);
  cout << b << " ";
  cout.width(8);
  cout << c << endl;
}
```

```c++
#include <iostream>
#include <format>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  cout << format("{:8} {:8} {:8}", a, b, c);
}
```

### 输出字符

![image-20241026210513112](./Typora图片/image-20241026210513112.png)

```c++
#include "stdio.h"
int main() {
  char c;
  scanf("%c", &c);
  printf("  %c\n", c);
  printf(" %c%c%c\n", c, c, c);
  printf("%c%c%c%c%c\n", c, c, c, c, c);
}
```

```c++
#include <iostream>
using namespace std;
int main() {
  char c;
  cin >> c;
  cout << "  " << c << endl;
  cout << " " << c << c << c << endl;
  cout << c << c << c << c << c << endl;
}
```

```C++
#include <iostream>
#include <format>
using namespace std;
int main() {
  char c;
  cin >> c;
  cout << format("  {0}", c) << endl; //指定输出后面第0位变量
  cout << format(" {0}{0}{0}", c) << endl;
  cout << format("{0}{0}{0}{0}{0}", c) << endl;
}
```

### 保留小数位数(float)

![image-20241027183837761](./Typora图片/image-20241027183837761.png)

```C++
#include "stdio.h"
int main() {
  float f;
  scanf("%f", &f);
  printf("%.3f\n", f);
}
```

默认情况下，`cout` 输出浮点数时可能会选择科学计数法或固定小数点表示法，具体取决于数值的大小。**只有使用 `fixed` 修饰符后**，`cout` 将始终以固定小数点表示法输出浮点数，并且小数点后的位数由 `precision` 控制。

```c++
#include <iostream>
using namespace std;
int main() {
  float f;
  cin >> f;
  cout.precision(3);
  cout << fixed;  
  cout << f << endl;
}
```

```C++
#include <iostream>
#include <format>
using namespace std;
int main() {
  float f;
  cin >> f;
  cout << format("{:.3f}", f) << endl;
}
```

### 保留小数位数(double)

读入一个双精度浮点数，保留 12位小数

![image-20241027185611550](./Typora图片/image-20241027185611550.png)

```C++
#include "stdio.h"
int main() {
  double d;
  scanf("%lf", &d);
  printf("%.12lf\n", d);
}
```

```C++
#include <iostream>
using namespace std;
int main() {
  double d;
  cin >> d;
  cout.precision(12);
  cout << fixed;
  cout << d << endl;
}
```

```c++
#include <iostream>
using namespace std;
#include <format>
int main() {
  double d;
  cin >> d;
  cout << format("{:.12f}\n", d);
}
```

### 综合输入

读入一个字符，一个整数，一个单精度浮点数，一个双精度浮点数，然后按顺序输出它们，并且要求在他们之间用一个空格分隔。输出浮点数时保留 66 位小数。

![image-20241029230920998](./Typora图片/image-20241029230920998.png)

```C++
#include "stdio.h"
int main() {
  char c;
  int i;
  float f;
  double d;
  scanf("%c%d%f%lf", &c, &i, &f, &d);
  printf("%c %d %.6f %.6lf\n", c, i, f, d);
}
```

```C++
#include <iostream>
using namespace std;
int main() {
  char c;
  int i;
  float f;
  double d;
  cin >> c >> i >> f >> d;
  cout.precision(6);
  cout << fixed;
  cout << c << ' ' << i << ' ' << f << ' ' << d << endl;
}
```

```C++
#include <iostream>
using namespace std;
#include <format>
int main() {
  char c;
  int i;
  float f;
  double d;
  cin >> c >> i >> f >> d;
  cout << format("{} {} {:.6f} {:.6f}",c, i, f, d);
}
```

### 浮点输出

读入一个双精度浮点数，分别按输出格式 `%f` ，`%f` 保留 5位小数，`%e` 和 `%g` 的形式输出这个数，每次在单独一行上输出。

**输入格式**

一个双精度浮点数。

**输出格式**

第一行是按 `%f` 输出的双精度浮点数；

第二行是按 `%f` 保留 55 位小数输出的双精度浮点数；

第三行是按 `%e` 输出的双精度浮点数；

第四行是按 `%g` 输出的双精度浮点数。

**输入**

```
12.3456789
```

**输出**

```
12.345679
12.34568
1.234568e+01
12.3457
```

```c++
#include "stdio.h"
#include <iostream>
using namespace std;
int main() {
  double d;
  cin >> d;
  printf("%f\n%.5f\n%e\n%g\n", d, d, d, d);
}
```

```C++
#include <iostream>
using namespace std;
int main() {
  double d;
  cin >> d;
  cout << fixed;
  cout << d << endl;
  cout.precision(5);
  cout << d << endl;
  cout.precision(6);
  cout << scientific;
  cout << d << endl;
  cout << defaultfloat;
  cout << d << endl;
}
```

```c++
#include <iostream>
#include <format>
using namespace std;
int main() {
  double d;
  cin >> d;
  cout << format("{0:f}\n{0:.5f}\n{0:e}\n{0:g}\n", d);
}
```

### 反向输出

![image-20241031014346147](./Typora图片/image-20241031014346147.png)

把三个字符挨个输入进来，在把三个字符挨个反向输出

```C++
#include <iostream>
using namespace std;
int main() {
  char a, b, c;
  cin >> a >> b >> c;
  cout << c << b << a << endl;
}
```

## 基础运算

![image-20241031015024962](./Typora图片/image-20241031015024962.png)

### 算数运算

![image-20241031015207369](./Typora图片/image-20241031015207369.png)

![image-20241031015409263](./Typora图片/image-20241031015409263.png)

### 自增自减

![image-20241031015541114](./Typora图片/image-20241031015541114.png)

### 赋值运算

![image-20241031015636219](./Typora图片/image-20241031015636219.png)

a的值一直在变化，也就是为什么a会被称为变量

![image-20241031015700011](./Typora图片/image-20241031015700011.png)

![image-20241031015811595](./Typora图片/image-20241031015811595.png)

### 总结

![image-20241031015833856](./Typora图片/image-20241031015833856.png)

## 实例

### 加法

![image-20241103000741699](./Typora图片/image-20241103000741699.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b;
  cin >> a >> b;
  cout << a + b << endl;
}
```

这里超过了int范围

![image-20241103001135629](./Typora图片/image-20241103001135629.png)

```c++
#include <iostream>
using namespace std;
int main() {
  long long a, b;
  cin >> a >> b;
  cout << a + b << endl;
}
```

### 乘法

int范围位21亿，无符号则扩展位两倍，注意这里的输出范围

![image-20241103001515779](./Typora图片/image-20241103001515779.png)

```c++
#include <iostream>
using namespace std;
int main() {
  unsigned int a, b;
  cin >> a >> b;
  cout << a * b << endl;
}
```

### 带余数除法

![image-20241103002923997](./Typora图片/image-20241103002923997.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b;
  cin >> a >> b;
  cout << a / b << ' ' << a % b << endl;
}
```

### 并联电阻阻值

![image-20241103211609627](./Typora图片/image-20241103211609627.png)

```C++
#include <iostream>
using namespace std;
int main() {
  double r1, r2;
  cin >> r1 >> r2;
  cout.precision(2);
  cout << fixed;
  cout << 1 / (1 / r1 + 1 / r2) << endl;
}
```

### 等差数列末项计算

![image-20241103212107066](./Typora图片/image-20241103212107066.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int a1, a2, n;
  cin >> a1 >> a2 >> n;
  int diff = a2 -a1;
  cout << a1 + diff * (n - 1) << endl;
}
```

## 类型转换

### 浮点型转整形

![image-20241104235803817](./Typora图片/image-20241104235803817.png)

### 不同大小浮点型互转

![image-20241104235858880](./Typora图片/image-20241104235858880.png)

![image-20241104235927606](./Typora图片/image-20241104235927606.png)

### 整形转浮点型

![image-20241105000328722](./Typora图片/image-20241105000328722.png)

### 整形之间转换

![image-20241105001253458](./Typora图片/image-20241105001253458.png)

### 强制转换表达式

![image-20241105001411793](./Typora图片/image-20241105001411793.png)

### 隐式转换级别

小于int的级别操作数进行运算时，都会先转化成int再进行运算

大于int级别操作数进行运算时，级别低的操作数数据类型先变得和级别搞得操作数数据类型一样再进行计算

级别高低：浮点大于整形，字节大的大于字节小的，无符号大于有符号

![image-20241105001610964](./Typora图片/image-20241105001610964.png)

## 实例

### 浮点转整形

![image-20241106005534743](./Typora图片/image-20241106005534743.png)

```c++
#include <iostream>
using namespace std;
int main() {
  double d;
  cin >> d;
  cout << (long long)d << endl;
}
```

### 字符转整形

![image-20241106010519386](./Typora图片/image-20241106010519386.png)

```c++
#include <iostream>
using namespace std;
int main() {
  char c;
  cin >> c;
  // cout << (int)c <<endl;
  
  int d = c;
  cout << d << endl;
}
```

### 整形转字符

![image-20241106010929972](./Typora图片/image-20241106010929972.png)

```
#include <iostream>
using namespace std;
int main() {
  int d;
  cin >> d;
  cout << (char)d << endl;
}
```

### 整形转布尔

```c++
#include <iostream>
using namespace std;
int main() {
  int d;
  cin >> d;
  cout << (bool)d << endl;
}
```

### 浮点相除取余

![image-20241107191813615](./Typora图片/image-20241107191813615.png)

```C++
#include <iostream>
using namespace std;
int main() {
  double a, b;
  cin >> a >> b;
  int k = a / b;
  cout << a - k * b << endl;
}
```

## 常用数学函数

![image-20241107192141808](./Typora图片/image-20241107192141808.png)

### 两点距离

![image-20241109210805292](./Typora图片/image-20241109210805292.png)

```c++
#include <iostream>
#include "math.h"
using namespace std;
int main() {
  double xa, ya, xb, yb;
  cin >> xa >> ya >> xb >> yb;
  cout.precision(3);
  cout << fixed;
  cout << sqrt(pow(xa - xb, 2) + pow(ya - yb, 2)) << endl;
}
```

### 三角形周长

![image-20241109222803895](./Typora图片/image-20241109222803895.png)

```c++
#include <iostream>
#include "math.h"
using namespace std;
int main() {
  double x1, y1, x2, y2, x3, y3;
  cin >> x1 >> y1 >> x2 >> y2 >> x3 >> y3;
  double a = hypot(x1 - x2, y1 - y2); //计算两点距离
  double b = hypot(x1 - x3, y1 - y3);
  double c = hypot(x2 - x3, y2 - y3);
  double p = (a + b + c) / 2;
  cout.precision(2);//保留两位小数
  cout << fixed;
  cout << sqrt(p * (p - a) * (p -b) * (p - c)) << endl;
}
```

### 次幂计算

![image-20241111012842461](./Typora图片/image-20241111012842461.png)

pow函数输出为浮点，需要进行类型转换

```c++
#include <iostream>
#include "math.h"
using namespace std;
int main() {
  int n;
  cin >> n;
  cout << (int)pow(2, n) << endl;
}
```

# 逻辑控制

## 逻辑相关运算

比较的时候，数据类型会转换成相同的类型，按照隐式转换级别来

### 比较

![image-20241112011717682](./Typora图片/image-20241112011717682.png)

![image-20241112011951249](./Typora图片/image-20241112011951249.png)

### 逻辑

![image-20241112012310786](./Typora图片/image-20241112012310786.png)

![image-20241112012355179](./Typora图片/image-20241112012355179.png)

![image-20241112012516605](./Typora图片/image-20241112012516605.png)

如果是true，以表达式一作为结果，如果是false，以表达式2作为结果

### 条件

1是odd，0是even

![image-20241112012941481](./Typora图片/image-20241112012941481.png)

## 实例

### 判断是否为两位数

```c++
#include <iostream>
using namespace std;
int main() {
  int x;
  cin >> x;
  cout << (x >= 10 && x <= 99) << endl;
}
```

### 逻辑或

如果你拥有 10 个印有“幸运”、或 20 个印有“鼓励”的瓶盖，就可以兑换一个神秘大奖。

```c++
#include <iostream>
using namespace std;
int main() {
  int x, y;
  cin >> x >> y;
  cout << (x >= 10 || y >= 20) << endl;
}
```

```c++
#include  <iostream>
using namespace std;
int main() {
  int w;
  cin >> w;
  cout << (w == 1 || w == 3 || w == 5? "NO" : "YES") << endl;
}
```



### 逻辑且

判断一个数能否同时被 3 和 5 整除

```c++
#include <iostream>
using namespace std;
int main() {
  int n ;
  cin >> n;
  cout << (n % 3 == 0 && n % 5 == 0 ? "YES" : "NO") << endl;
}
```

## 条件控制

### if

![image-20241116232418248](./Typora图片/image-20241116232418248.png)

### 作用域

![image-20241116232628245](./Typora图片/image-20241116232628245.png)

### IF初始化

在条件判断前，可以写一个初始化语句，其中的值可以再分支语句中使用

![image-20241116233312039](./Typora图片/image-20241116233312039.png)

![image-20241116233245912](./Typora图片/image-20241116233245912.png)

### 书写规范

<img src="./Typora图片/image-20241116233345457.png" alt="image-20241116233345457" style="zoom: 50%;" />

## 实例

### 判断正负数

给定一个整数 N*N*，判断其正负。如果 N>0*N*>0, 输出 `positive` ; 如果 N=0*N*=0, 输出 `zero` ; 如果 N<0,*N*<0, 输出 `negative`。

```c++
#include <iostream>
using namespace std;
int  main() {
  int n;
  cin >> n;
  if (n > 0) cout << "positive" << endl;
  else if (n == 0) cout << "zero" << endl;
  else cout << "negative" << endl;
}
```

条件互斥的情况下全部用if也行

```
  if (n > 0) cout << "positive" << endl;
  if (n == 0) cout << "zero" << endl;
  if (n < 0) cout << "negative" << endl;
```

### 整数大小比较

输入两个整数，比较它们的大小。若 x>y*x*>*y* ，输出 `>` ；若 x=y*x*=*y* ，输出 `=` ；若 x<y*x*<*y*，输出 `<` 。

```c++
#include <iostream>
#include "stdio.h"
using namespace std;
int main() {
  unsigned int x;
  int y;
  cin >> x >> y;
  if (y > 0) puts(">");
  else {
    if (x < y) puts("<");
    if (x == y) puts("=");
    if (x > y) puts(">");
  }
}
```

### 最大数输出

输入三个整数，输出最大的数。

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  int max = a;
  if (b > max) max = b;
  if (c > max) max = c;
  cout << max << endl;
}
```

### 判断能否被 3，5，7 整除

给定一个整数 x*x*，判断它能否被 33，55，77 整除，并输出以下信息：

1、能同时被 3,5,73,5,7 整除（直接输出 `3 5 7`，每个数中间一个空格）；

2、只能被其中两个数整除（按从小到大的顺序输出两个数，例如：`3 5` 或者 `3 7` 或者 `5 7`，中间用空格分隔）；

3、只能被其中一个数整除（输出这个除数）；

4、不能被任何数整除，输出小写字符 `n`。

```C++
#include <iostream>
#include "stdio.h"
using namespace std;
int main() {
  int n;
  cin >> n;
  // 3 5 7 之间互质，最小公约数为其乘积
  if (n % 105 == 0) puts("3 5 7");
  else if (n % 15 == 0) puts("3 5");
  else if (n % 21 == 0) puts("3 7");
  else if (n % 35 == 0) puts("5 7");
  else if (n % 3 == 0) puts("3");
  else if (n % 5 == 0) puts("5");
  else if (n % 7 == 0) puts("7");
  else puts("n");
}
```

**也可以看成能否被三个数从小到大整除**

```c++
#include <iostream>
#include "stdio.h"
using namespace std;
int main() {
  int n;
  cin >> n;
  if (n % 3 && n % 5 && n % 7) puts("n");
  else {
    if (n % 3 == 0) cout << "3 ";
    if (n % 5 == 0) cout << "5 ";
    if (n % 7 == 0) cout << "7 ";
  }
  cout << endl;
}
```

### 判断恰好有一门课不及格

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  int cnt = 0;
  if (a < 60) cnt ++;
  if (b < 60) cnt ++;
  if (c < 60) cnt ++;
  cout << (cnt == 1) << endl;
}
```

### 用时判断

在清华校园里，没有自行车，上课办事会很不方便。但实际上。并非去办任何事情都是骑车快，因为骑车总要找车、开锁、停车、锁车等，这要耽误一些时间。假设找到自行车，开锁，上车起步的时间为 2727 秒；停车锁车的时间为 2323 秒；步行每秒行走 1.2 米，骑车每秒行走 3.0 米。请判断走不同的距离去办事，是骑车快还是走路快。

```c++
#include <iostream>
using namespace std;
int main() {
  int dis;
  cin >> dis;
  if (double walkT = dis / 1.2, bikeT = dis / 3 + 50; walkT < bikeT ) {
    puts("Walk");
  } else if (walkT == bikeT) {
    puts("All");
  } else {
    puts("Bike");
  }
}
```

### 分段函数

![image-20241120004551265](./Typora图片/image-20241120004551265.png)

```C++
#include <iostream>
using namespace std;
int main() {
  cout.precision(3);
  cout << fixed;
  double x;
  cin >> x;
  if (x < 5) {
    cout << -x + 2.5 << endl;
  } else if (x < 10) {
    cout <<  2 - 1.5 * (x - 3) * (x - 3) << endl;
  } else {
    cout << x / 2 - 1.5 << endl;
  }
}
```

### 解二元一次方程组

![image-20241120232104177](./Typora图片/image-20241120232104177.png)

```c++
#include <iostream>
#include "math.h"
#include <format>
#include <algorithm>
using namespace std;
int main() {
  double a, b, c;
  cin >> a >> b >> c;
  if (double rt = b * b - 4 * a * c; rt < 0) {
    cout << "No answer!" << endl;
  } else if (rt > 0) {
    rt =  sqrt(rt);
    double x1 = (-b - rt) / a / 2;
    double x2 = (-b + rt) / a / 2;
    cout << format("x1={:.5f};x2={:.5f}\n", min(x1, x2), max(x1, x2));
  } else {
    cout << format("x1=x2={:.5f}\n", -b / a / 2);
  }
}
```

## 循环控制

### for

先执行循环体，再改变循环条件

![image-20241120233455863](./Typora图片/image-20241120233455863.png)

![image-20241120233611531](./Typora图片/image-20241120233611531.png)

### while

![image-20241120233740889](./Typora图片/image-20241120233740889.png)

![image-20241120233806794](./Typora图片/image-20241120233806794.png)

### do..while

会先进行循环内的操作，在判断是否继续循环

![image-20241120233853600](./Typora图片/image-20241120233853600.png)

### break

![image-20241120234117299](./Typora图片/image-20241120234117299.png)

### continue

跳过后续还未执行的循环体，开启下一轮循环

![image-20241120234221690](./Typora图片/image-20241120234221690.png)

### goto

程序跳到标签位置，执行后续的代码

![image-20241120234332935](./Typora图片/image-20241120234332935.png)

### switch

![image-20241125182425618](./Typora图片/image-20241125182425618.png)

![image-20241125182649398](./Typora图片/image-20241125182649398.png)

分支越多，switch运行越快，因为在汇编层面，if else是一个一个进行比较，而switch使用二分方式比较

在汇编语言中，dl代表c

if else中就是一个一个比

![image-20241125183211867](./Typora图片/image-20241125183211867.png)

switch这里，小于45，直接在42和43之间比

![image-20241125183040089](./Typora图片/image-20241125183040089.png)

## 实例

### 输入5个数取最大值

![image-20241121221744999](./Typora图片/image-20241121221744999.png)

```c++
#include <iostream>
#include <algorithm>
using namespace std;
int main() {
  int n;
  cin >> n;
  int maxx = 0;
  while (n--) {
    int x;
    cin >> x;
    maxx = max(maxx, x);
  }
}
```

### 条件嵌套

![image-20241122193454392](./Typora图片/image-20241122193454392.png)

```C++
#include <iostream>
using namespace std;
int main() {
  int m, n;
  cin >> m >> n;
  int cnt = 0;
  while (n--) {
    int x;
    cin >> x;
    if (x > m) cnt++;
    else m -= x;
  }
  cout << cnt << endl;
}
```

### 余数相同问题

![image-20241122194331293](./Typora图片/image-20241122194331293.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b, c;
  cin >> a >> b >> c;
  for (int x = 2;; x++) {
    if (a % x == b%x && b % x == c % x) {
      cout << x << endl;
      break;
    }
  }
}
```

### 次幂的末尾数

幂 *a**b* 的末 3 位数是多少？

输入格式

两个正整数 a*a*，b*b*。1≤a≤100，1≤b≤10000。

输出格式

从高位到低位输出幂的末三位数字，中间无分隔符。若幂本身不足三位，在前面补零。

输入输出样例

输入#1

```
2 3
```

输出#1

```
008
```

输入#2

```
7 2011
```

输出#2

```
743
```

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b;
  cin >> a >> b;
  int res = 1;
  while (b--){
    res *= a;
    res %= 1000;
  }
  cout.width(3);
  cout.fill('0');
  cout << res << endl;
}
```

- **`cout.width(3);`**：这行代码设置了后续输出操作的最小字段宽度为3。这意味着输出的结果占据至少3个字符的位置。
- **`cout.fill('0');`**：这行代码设置了填充字符为 '0'。当输出结果的实际字符数少于设定的宽度（3个字符）时，系统会用 '0' 来填充空缺的位置在数字的左侧。

### 循环嵌套

![image-20241124113551348](./Typora图片/image-20241124113551348.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int n;
  cin >> n;
  double e = 1;
  for (int i = 1; i <= n; i++) {
    double res = 1;
    for (int j = 1; j <= i; j++) {
      res *= j;
    }
    e += 1 / res;
  }
  cout << fixed;
  cout.precision(10);
  cout << e << endl;
}
```

### 与7无关的数

![image-20241124114829139](./Typora图片/image-20241124114829139.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int n;
  cin >> n;
  int sum =0;
  for (int i = 1; i <= n; i++) {
    if (i % 7 == 0) continue;
    bool seven = false;
    int m = i;
    while (m){
      int digit = m % 10;
      if (digit == 7) {
        seven = true;
        break;
      }
      m /= 10;
    }
    if (seven) continue;
    sum += i * i;
  }
  cout << sum << endl;
}
```

### 第n个质数

![image-20241124120143310](./Typora图片/image-20241124120143310.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int n;
  cin >> n;
  int cnt = 0;
  for (int x = 2;;x++) {
    bool isPrime = true;
    for(int i = 2; i < x; i++) {
      if (x % i == 0) {
        isPrime = false;
        break;
      }
    }
    if (isPrime) {
      cnt ++;
      if (cnt == n) {
        cout << x << endl;
        break;
      }
    }
  }
}
```

### switch简单计算器

![image-20241125184528446](./Typora图片/image-20241125184528446.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int a, b;
  char c;
  cin >> a >> b >> c;
  switch (c){
  case '+':
    cout << a + b << endl;
    break;
  case '-':
    cout << a - b << endl;
    break;
  case '*':
    cout << a * b << endl;
    break;
  case '/':
    if (b == 0) cout << "Divided by zero!" << endl;
    else cout << a / b << endl;
    break;
  default:
    cout << "Invalid opeartor" << endl;
    break;
  }
}
```

### a的b次方是周几

![image-20241125185425162](./Typora图片/image-20241125185425162.png)

```C++
#include <iostream>
using namespace std;
int main() {
  int a, b;
  cin >> a >> b;
  int res = 1;
  while (b--){
    res *= a;
    res %= 7;
  }
  switch (res){
  case 0:
    cout << "Sunday" << endl;
    break;
  case 1:
    cout << "Monday  " << endl;
    break;
  case 2:
    cout << "Tuesday  " << endl;
    break;
  case 3:
    cout << "Wednesday  " << endl;
    break;
  case 4:
    cout << "Thursday " << endl;
    break;
  case 5:
    cout << "Friday  " << endl;
    break;
  default:
  cout << "Saturday " << endl;
    break;
  }
}
```

# 数组

存储相同数据类型元素的集合

![image-20241126193914360](./Typora图片/image-20241126193914360.png)

## 定义数组

![image-20241126194016880](./Typora图片/image-20241126194016880.png)

## 使用

![image-20241126194433926](./Typora图片/image-20241126194433926.png)

### 输入输出

![image-20241126194620109](./Typora图片/image-20241126194620109.png)

另外一种输入方式

![image-20241127193823924](./Typora图片/image-20241127193823924.png)

### 初始化

![image-20241126194906337](./Typora图片/image-20241126194906337.png)

![image-20241126195204394](./Typora图片/image-20241126195204394.png)

### 多维数组

![image-20241126195259788](./Typora图片/image-20241126195259788.png)

![image-20241126195314038](./Typora图片/image-20241126195314038.png)

## 实例

### 一圈小朋友分糖果

![image-20241127195458749](./Typora图片/image-20241127195458749.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int arr[5];
  for (auto& x: arr) cin >> x;
  int sum = 0;
  for (int i = 0; i < 5; i ++) {
    sum += arr[i] % 3;
    arr[(i+4)%5] += arr[i] / 3;
    arr[(i+1)%5] += arr[i] / 3;
    arr[i] /= 3;
  }
  for (auto& x: arr) cout << x << ' '; cout << endl;
  cout << sum << endl;
}
```

### 计算书费

![image-20241127200605265](./Typora图片/image-20241127200605265.png)

```c++
#include <iostream>
using namespace std;
int main() {
  double prices[] = {28.9, 32.7, 45.6, 78, 35, 86.2, 27.8, 43, 56, 65};
  double sum = 0;
  // 关键字 auto 自动推断变量的类型,符号 & 表示 price 是对 prices 数组中元素的引用
  for (auto& price: prices) {
    int cnt;
    cin >> cnt;
    sum += price * cnt;
  }
  cout << fixed;
  cout.precision(1);
  cout << sum << endl;
}
```

### e的值计算

在数组中提前计算所有要算的值，拆开双循环

<img src="./Typora图片/image-20241128190129505.png" alt="image-20241128190129505" style="zoom:67%;" />

原本

```c++
#include <iostream>
using namespace std;
int main() {
  int n;
  cin >> n;
  double e = 1;
  for (int i = 1; i <= n; i++) {
    double res = 1;
    for (int j = 1; j <= i; j++) {
      res *= j;
    }
    e += 1 / res;
  }
  cout << fixed;
  cout.precision(10);
  cout << e << endl;
}
```

优化

```c++
#include <iostream>
using namespace std;
int main() {
  double Fact[16];
  Fact[0] = 1;
  for (int i = 1; i < 16; i++) {
    Fact[i] = Fact[i-1] / i;
  }
  int n;
  cin >> n;
  double e = 0;
  for (int i = 0; i <= n; i++) {
    e += Fact[i];
  }
  cout << fixed;
  cout.precision(10);
  cout << e << endl;
}
```

### 数组逆序重存放

![image-20241128191601091](./Typora图片/image-20241128191601091.png)

数组前后对称交换变量

```
#include <iostream>
using namespace std;
int main() {
  int n;
  cin >> n;
  int arr[n];
  for (auto& x: arr) cin >> x;
  for (int i = 0, j = n - 1; i < j; i++, j--) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
  }
  for (auto x: arr) cout << x << ' ';
  cout << endl;
}
```

### 计算矩阵边缘元素之和

![image-20241129154101028](./Typora图片/image-20241129154101028.png)

```c++
#include <iostream>
using namespace std;
int main() {
  int n, m;
  cin >> n >> m;
  int a[n][m];
  for (auto& line: a) {
    for(auto& x: line) {
      cin >> x;
    }
  }
  int sum = 0;
  for (int i = 0; i < n; i++) {
    sum += a[i][0];
    if (m - 1 == 0) continue;
    sum += a[i][m-1];
  }
  for (int j = 1; j < m - 1; j++) {
    sum += a[0][j];
    if (n - 1 == 0) continue;
    sum += a[m-1][j];
  }
  cout << sum << endl;
}
```

### 行的最大值，列的最小值

给定一个 5×5 的矩阵，每行只有一个最大值，每列只有一个最小值，寻找这个矩阵的鞍点。鞍点指的是矩阵中的一个元素，它是所在行的最大值，并且是所在列的最小值。

例如：在下面的例子中，第 4 行第 1 列的元素就是鞍点，值为 8。

```
11 3 5 6 9
12 4 7 8 10
10 5 6 9 11
8  6 4 7 2
15 10 11 20 25
```

```c++
#include <iostream>
using namespace std;
int main() {
  int n =5, m = 5;
  int a[n][m];
  for(auto& line: a) {
    for (auto& x: line) {
      cin >> x;
    }
  }
  bool found = false;
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      bool isMax = true, isMin = true;
      for (int k = 0; k < m; k++) {
        if (a[i][k] > a[i][j]) {
          isMax = false;
          break;
        }
      }
      for (int k = 0; k < m; k++) {
        if (a[k][j] < a[i][j]) {
          isMin = false;
        }
      }   
      if (isMax && isMin) {
        cout << i + 1 << ' ' << j + 1 << ' ' << a[i][j] << endl;
        found = true;
      }
    }
  }
  if (!found) cout << "not found" << endl;
}
```

### 矩阵转置

![image-20241201171233513](./Typora图片/image-20241201171233513.png)

```C++
#include <iostream>
using namespace std;
int main() {
  int n, m;
  cin >> n >> m;
  int a[n][m], b[m][n];
  for (auto& line: a) {
    for (auto& x: line) {
      cin >> x;
    }
  }
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      b[j][i] = a[i][j];
    }
  }
  for (auto& line: b) {
    for (auto& x: line) {
      cout << x << ' ';
    }
    cout << endl;
  }
}
```

### 矩阵旋转90°

![image-20241201172052839](./Typora图片/image-20241201172052839.png)

根据几个行列变化例子，找到矩阵每个值前后对应位置规律

```c++
#include <iostream>
using namespace std;
int main() {
  int n, m;
  cin >> n >> m;
  int a[n][m], b[m][n];
  for (auto& line: a) {
    for (auto& x: line) {
      cin >> x;
    }
  }
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
      // (0,0) -> (0, n-1)
      // (0,m-1) -> (m-1,n-1)
      // (n-1,0) -> (0,0)
      b[j][n-1-i] = a[i][j];
    }
  }
  for (auto& line: b) {
    for (auto& x: line) {
      cout << x << ' ';
    }
    cout << endl;
  }
}
```

## vector

动态数组容器

**一开始不用确定数组大小，可以动态调整数组容量**

### 定义方式

![image-20241202220728530](./Typora图片/image-20241202220728530.png)

index(n) 指定了外部向量 index 的大小为 n，这意味着 index 将包含 n 个元素，每个元素都是一个 vector<int>。这些元素可以被视为二维数组的行，因此行数是确定的，为 n

![image-20241210211754236](./Typora图片/image-20241210211754236.png)

### 取值

at会监测索引是否越界

![image-20241202220827408](./Typora图片/image-20241202220827408.png)

### 输入输出

![image-20241202220940037](./Typora图片/image-20241202220940037.png)

### 初始化

![image-20241202221104833](./Typora图片/image-20241202221104833.png)

### 赋值和比较

![image-20241202221309269](./Typora图片/image-20241202221309269.png)

### 常用函数

![image-20241202221758606](./Typora图片/image-20241202221758606.png)

## 实例

### [语言月赛202304] 你的牌太多了

![image-20241207201219650](./Typora图片/image-20241207201219650.png)

**输出格式**

输出一行一个整数，表示坏蛋小 F 结束时手里剩余的牌数。

输入输出样例

**输入#1**

```
3 1 2
1 1 1
1 2 1
1 1 1
2 2 1
2 3 1
```

**输出#1**

```
1
```

**输入#2**

```
3 2 2
1 2 1
1 1 1
1 2 1
2 2 2
1 2 3
```

**输出#2**

```
0
```

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
  int n, m, r;
  cin >> n >> m >> r;
  vector<int> f1(n), f2(n), p1(n), p2(n);
  for (auto& x: f1) cin >> x;
  for (auto& x: p1) cin >> x;
  for (auto& x: f2) cin >> x;
  for (auto& x: p2) cin >> x;
  while (n--){
    int order;
    cin >> order;
    order--;
    int index = -1;
    for (int i = 0; i < f2.size(); i++) {
      if (f2[i] == f1[order] && p2[i] > p1[order]) {
        if (index == -1 || p2[i] < p2[index]) {
          index = i;
        }
      }
    }
    if (index != -1) {
      f2.erase(f2.begin() + index);
      p2.erase(p2.begin() + index);
    }
  }
  cout << f2.size() << endl;
}
```

### 取硬币博弈问题

![image-20241208180620537](./Typora图片/image-20241208180620537.png)

输入输出样例

输入#1

```
2
3 2
```

输出#1

```
2 3
```

输入#2

```
5
1 2 3 4 5
```

输出#2

```
9 6
```

```c++
#include<iostream>
#include <vector>
using namespace std;
int main() {
  int n;
  cin >> n;
  vector<long long> coins(n);
  for (auto& x: coins) cin >> x;
  vector<vector<long long>> players(2);
  int currentPlayer = 0;
  while (!coins.empty()) {  //桌面上硬币不为空
    auto& player = players[currentPlayer];  //代表当前是哪个玩家的回合
    currentPlayer = (currentPlayer + 1) % 2;

    // 玩家牌里点数总和
    long long sum = 0;
    for (auto& x: player) sum += x;

    // 每位玩家应该选择桌面上的哪一张牌
    int index = -1;  //设置为-1，表示没有找到合适的硬币
    int mini = 0;

    // 选择面值不超过当前自己钱包中硬币的总面额
    // 的硬币中面额最大的一枚硬币
    for (int i = 0; i < coins.size(); i++) {
      if (coins[i] <= sum) {
        if (index == -1 || coins[i] > coins[index]) {
          index = i;
        }
      }
      // 找到桌面上最小的一张牌的位置
      if (coins[i] < coins[mini]) {
        mini = i;
      }
    }
    if (index != -1) {
      player.push_back(coins[index]);
      coins.erase(coins.begin() + index);
    } else {
      player.push_back(coins[mini]);
      coins.erase(coins.begin() + mini);
    }
  }
  for (auto& player: players) {
    long long sum = 0;
    for (auto& x: player) sum += x;
    cout << sum << ' ';
  }
  cout << endl;
}
```

### 最小负载分配问题

![image-20241210212356306](./Typora图片/image-20241210212356306.png)

```c++
#include<iostream>
#include <vector>
using namespace std;
int main() {
    int n, m;
    cin >> n >> m;
    vector<vector<int>> indexes(n);
    vector<long long> sum(n);
    for (int index = 1; index <= m; index++) {
        int t;
        cin >> t;
        int candidate = 0;
        for (int i = 1; i < n; i++) {
            if (sum[i] < sum[candidate]) {
                candidate = i;
            }
        }
        sum[candidate] += t;
        indexes[candidate].push_back(index);
    }
    for (auto& line : indexes) {
        if (line.size() == 0) {
            cout << 0 << endl;
        } else {
            for (auto& index : line) {
                cout << index << ' ';
            }
            cout << endl;
        }
    }
}
```

# C语言字符串

字符串就是带有终止字符的字符数组

## 初始化

末尾有\0表示终止字符

![image-20241211091858230](./Typora图片/image-20241211091858230.png)

转义字符，以双引号举例

添加斜杠转义

![image-20241211092009758](./Typora图片/image-20241211092009758.png)

## 输入输出

**读取**

![image-20241211092458562](./Typora图片/image-20241211092458562.png)

![image-20241211093028625](./Typora图片/image-20241211093028625.png)



**输出**

会挨个输出字节，直到遇到终止字符

![image-20241211092726761](./Typora图片/image-20241211092726761.png)

## 常用函数

![字符串](./Typora图片/字符串.png)

字符转化为整形，long long，double

![image-20241211095538410](./Typora图片/image-20241211095538410.png)

## 判断字符类型

表中如isspace就只能判断是否是空格

![ctype](./Typora图片/ctype.png)

## 实例

### 字符个数

输入一行字符，统计出其中数字字符的个数。

输入格式

一行字符串，总长度不超过 255。

输出格式

输出为 1 行，输出字符串里面数字字符的个数。

```c++
#include <iostream>
#include <string.h>
using namespace std;
int main() {
  char str[256];
  cin.getline(str, 256);
  int cnt = 0;
  for (size_t i = 0; i < strlen(str); i++) {
    cnt += isdigit(str[i]);
  }
  cout << cnt << endl;
}

```

### 字符 ASCII 值相加

将字符单个切片取出就能得到字符串中每个字符的ascll码

亲朋字符串定义如下：给定字符串 s*s* 的第一个字符的 ASCII 值加第二个字符的 ASCII 值，得到第一个亲朋字符；s*s* 的第二个字符加第三个字符得到第二个亲朋字符；依此类推。注意：亲朋字符串的最后一个字符由 s*s* 的最后一个字符 ASCII 值加 s*s* 的第一个字符的 ASCII 值。

**输入格式**

输入一行，一个长度大于等于 22，小于等于 100100 的字符串。

**输出格式**

输出一行，为变换后的亲朋字符串。输入保证变换后的字符串只有一行。

**输入输出样例**

输入#1

```
1234
```

输出#1

```
cege
```

```c++
#include <iostream>
#include <string.h>
using namespace std;
int main() {
  char digits[101], str[101];
  cin >> digits;
  int len = strlen(digits);
  for (int i = 0; i < len; i++) {
    str[i] = digits[i] + digits[(i+1)%len];
  }
  str[len] = 0;
  cout << str << endl;
}
```

### 字母移动固定的位数

![image-20241216215638815](./Typora图片/image-20241216215638815.png)

```c++
#include <iostream>
#include <string.h>
#include <ctype.h>
using namespace std;
int main() {
  char str[10001];
  cin.getline(str, 10001);
  for (int i = 0; i < strlen(str); i++) {
    if (isupper(str[i])) {
      // 减去A方便把ascll码归零取模
      str[i] = (str[i] - 'A' + 25) % 26 + 'A';
    }
    if (islower(str[i])) {
      str[i] = (str[i] - 'a' + 25) % 26 + 'a';
    }
  }
  cout << str << endl;
}
```

### 删除指定后缀

给定一个单词，如果该单词以 er 、 ly 或者 ing 后缀结尾，则删除该后缀

输入#1

```
referer
```

输出#1

```
refer
```

```c++
#include <iostream>
#include <string.h>
using namespace std;
int main() {
  char str[33];
  cin >> str;
  int len = strlen(str);
  if (len >= 2 && strcmp(str + len - 2, "er") == 0) {
    str[len-2] = 0;
  } else if (len >= 2 && strcmp(str + len - 2, "ly") == 0) {
    str[len-2] = 0;
  } if (len >= 3 && strcmp(str + len - 3, "ing") == 0) {
    str[len-3] = 0;
  }
  cout << str << endl;
}
```

### 分割单词

![image-20241218221334895](./Typora图片/image-20241218221334895.png)

```c++
#include<iostream>
#include <string.h>
using namespace std;
int main() {
  char str[30000];
  cin.getline(str, 30000);
  char delimiter[] = " ,.";
  auto token = strtok(str, delimiter);
  char shortest[101], longest[101];
  strcpy(shortest, token);
  strcpy(longest, token);
  while (token) {
    if (strlen(token) < strlen(shortest)) {
      strcpy(shortest, token);
    }
    if (strlen(token) > strlen(longest)) {
      strcpy(longest, token);
    }
    token = strtok(NULL, delimiter);
  }
  cout << longest << endl;
  cout << shortest << endl;
}
```

# string(C++字符串）

## 初始化

str6从第二位开始初始化，str7从第二位开始初始化,初始化两位

![image-20241219232948902](./Typora图片/image-20241219232948902.png)

## 输入输出

![image-20241220003824907](./Typora图片/image-20241220003824907.png)

## 操作

支持的运算

![image-20241220004217160](./Typora图片/image-20241220004217160.png)

## 常用函数

[std::basic_string - cppreference.com](https://zh.cppreference.com/w/cpp/string/basic_string)

![image-20241220004819367](./Typora图片/image-20241220004819367.png)

## 实例

### 压缩编码

给定一个完全由数字字符(0,1',2…’9’)构成的字符串 str，请写出str的p型编码串。例如:字符串 122344111 可被描述为 1个1、2个2、1个3、2个4、3个1 ，因此我们说 122344111 的p型编码串为 1122132431 ;类似的道理，编码串 101 可以用来描述 1111111111;00000000000 可描述为 11个0，因此它的p型编码串即为110:100200300可描述为 1个1、2个0、1个2、2个0、1个3、2个0，因此它的p型编码串为112012201320。

**输入格式**

输入仅一行，包含字符串 str。每一行字符串最多包含 10001000 个数字字符。

**输出格式**

输出该字符串对应的 p*p* 型编码串。

**输入输出样例**

输入#**1**

```
122344111
```

输出#**1**

```
1122132431
```

```c++
#include <iostream>
#include <string>
using namespace std;
int main() {
  string str, res;
  cin >> str;
  char lastChar = str[0];
  int length = 1;
  for (int i = 1; i <= str.length(); i++) {
    if (str[i] == lastChar) {
      length++;
    } else {
      res += to_string(length) + lastChar;
      lastChar = str[i];
      length = 1;
    }
  }
  cout << res << endl;
}
```

### 子串个数

luoluoguluogu，luogu字前面字符串中出现了几次

```c++
#include <iostream>
#include <string>
using namespace std;
int main() {
  string str;
  cin >> str;
  int index = str.find("luogu");
  int cnt = 0;
  while (index != -1)
  {
    cnt ++;
    index = str.find("luogu", index + 1);
  }
  cout << cnt << endl;
}
```

### 提取方程系数

提取里面每个方程的系数,类似下面这种

2x+4=10 

-3x+13=10 

4x-8=16

```c++
#include <iostream>
#include <string>
#include <vector>
using namespace std;
int main() {
  int n, q;
  cin >> n >> q;
  vector<int> a, b, c;
  while (n--)
  {
    string str;
    cin >> str;
    int plus = str.find("x");
    int equal = str.find("=");
    a.push_back(stoi(str.substr(0, plus)));
    b.push_back(stoi(str.substr(plus + 1, equal - plus - 1)));
    c.push_back(stoi(str.substr(equal + 1)));
  }
 }
```

### 多行字符串输入合并

```c++
string game, str;
while (cin >> str) {
game += str;
}
```



# 函数

## 定义

![image-20241228104924629](./Typora图片/image-20241228104924629.png)

## 传值

形参可设置默认值：![image-20241228105130656](./Typora图片/image-20241228105130656.png)

但不可以前面有默认值，后面没有，因为调用函数时，传参按照顺序来的

有的函数没有返回值，可以把返回值定义为void，表示空

![image-20241228105439473](./Typora图片/image-20241228105439473.png)



函数的形参必须加上引用符号&，才能对传入函数的实参进行修改，否则仅仅是函数内部变量的修改

![image-20241228105708087](./Typora图片/image-20241228105708087.png)



尽可能用引用来传递值，因为这样代表同一个地址，如果直接把外面的值传递给函数的参数，相当于新建了一个同样大小的数据

值传递中，函数接收的是参数的一个副本。当你在函数内部修改参数时，实际修改的是副本，而不是原始变量。

![image-20241228105904820](./Typora图片/image-20241228105904820.png)



数组在传递给函数的过程中，由数组转化成了指针，数组传递给函数的语法：

![image-20241228110517726](./Typora图片/image-20241228110517726.png)

## 总结

![image-20241228110622000](./Typora图片/image-20241228110622000.png)

## 实例

### 不同分制比赛结果输出

![image-20250102130119839](./Typora图片/image-20250102130119839.png)

**输入输出样例**

**输入#1**

```
WWWWWWWWWWWWWWWWWWWW
WWLWE
```

**输出#1**

```
11:0
11:0
1:1

21:0
2:1
```

```c++
#include <iostream>
#include <cstdlib>
using namespace std;
void getScore(string &game, int limit) {
  int win = 0, lose = 0;
  for (char &c : game) {
    if ( c == 'E') {
      cout << win << ':' << lose << endl;
      return;
    } else if ( c == 'W') {
      win ++;
    } else {
      lose ++;
    }
    if (abs(win - lose) >= 2 && (win >= limit || lose >= limit)) {
      cout << win << ':' << lose << endl;
      win = lose = 0;
    }
  }
}
int main() {
  string game, str;
  while (cin >> str) {
    game += str;
  }
  getScore(game, 11);
  cout << endl;
  getScore(game, 21);
}
```

### 可能得到的最大分数

举穷所有情况，取得分最大的那一种

**[CRCI2008-2009] MAJSTOR**

**题目背景**

剪刀石头布是一种流行的两人游戏。

**题目描述**

在游戏中，两人都用自己的手出示三个手势之一：石头，布或剪刀。 

如果两个玩家出示相同的手势，则该游戏为平局。否则，剪刀胜布，布胜石头，而石头胜剪刀。

Sven 多年来一直在研究游戏的心理错综复杂性，并已成为游戏的真正大师，他的朋友们在一对一的游戏中没有与他抗衡的机会。

随着世界锦标赛的临近，Sven 正在与 $n$ 个朋友同时玩游戏，以练习他的技能。 

游戏有 $r$ 轮，在每一轮中，Sven 和他的每个朋友出三种手势之一。

计分规则：

- Sven 获胜，得 $2$ 分。
- 平局，得 $1$ 分。
- 对手获胜，得 $0$ 分。

编写一个程序来计算 Sven 的总得分，如果他事先知道他的朋友会出的所有手势，那么他最大得分也是可能的。

**输入格式**

第一行，一个正整数 $r$，表示比赛轮数。

第二行，$r$ 个字符，第 $i$ 个字符表示 Sven 在第 $i$ 局出的手势，只可能是 `S`（剪刀），`P`（布），`R`（石头）。

第三行，一个正整数 $n$，表示他的朋友数。

接下来 $n$ 行，每行都有 $r$ 个字符，第 $j$ 个字符表示 Sven 的第 $i$ 个朋友在第 $j$ 局出的手势，只可能是 `S`（剪刀），`P`（布），`R`（石头）。

**输出格式**

第一行，输出 Sven 的实际分数。

假设他的朋友没有改变他们的手势，则在第二行输出他最大的得分。

**样例 #1**

**样例输入 #1**

```
5
SSPPR
1
SSPPR
```

**样例输出 #1**

```
5
10
```

样例 #2

样例输入 #2

```
5
SSPPR
2
PPRRS
RRSSP
```

样例输出 #2

```
10
15
```

样例 #3

样例输入 #3

```
4
SPRS
4
RPRP
SRRR
SSPR
PSPS
```

样例输出 #3

```
12
21
```

**结果代码**

```c++
#include <iostream>
#include <vector>
using namespace std;
int judge(char a, char b) {
  if (a == b) return 1;
  if (a == 'S' && b == 'P') return 2;
  if (a == 'P' && b == 'R') return 2;
  if (a == 'R' && b == 'S') return 2;
  return 0;
}
int main() {
  string SPR = "SPR";  //所有手势字符串，用于实验最高分
  int n, m;
  cin >> m;  //比赛轮数
  string mine;  
  cin >> mine;  // 出的手势
  cin >> n;
  vector<string> players(n);
  for (auto &player : players) cin >> player;  // 对手手势
  int ans1 = 0, ans2 = 0;
  for (int i = 0; i < m; i++) {
    vector<int> scores(3);
    for (auto &player : players) {
      ans1 += judge(mine[i], player[i]);
      for (int j = 0; j < 3; j++) {
        scores[j] += judge(SPR[j], player[i]);  //每个手势在每一轮得到的分数存储起来
      }
    }
    ans2 += max(scores[0], max(scores[1], scores[2]));  //取最大手势得分
  }
  cout << ans1 << endl << ans2 << endl;
}
```

## main函数

![image-20250105122734375](./Typora图片/image-20250105122734375.png)

### 1. `int argc`

- **含义**: `argc` 表示“argument count”，即程序启动时传递给 `main` 函数的参数个数。
- **用途**: 它告诉程序有多少个命令行参数被传递，包括程序本身的名称。例如，如果你在命令行中运行程序时输入 `./my_program 5 10 15`，那么 `argc` 的值将是 `4`（程序名称 `./my_program` 和三个参数 `5`、`10`、`15`）。

### 2. `char** argv`

- **含义**: `argv` 表示“argument vector”，它是一个指向字符指针数组的指针。这个数组包含了所有的命令行参数，每个参数都是一个 C 风格的字符串（字符数组）。
- **用途**: `argv` 数组中的每个元素都是一个字符串，代表从命令行传入的参数。`argv[0]` 通常是程序的名称，而后续的元素（`argv[1]`、`argv[2]` 等）是用户输入的其他参数。

### 例子

对于命令行输入：

```
复制./my_program 5 10 15
```

- `argc` 的值将会是 `4`。

- ```
  argv
  ```

   

  数组的内容将会是：

  - `argv[0]` = `"./my_program"`
  - `argv[1]` = `"5"`
  - `argv[2]` = `"10"`
  - `argv[3]` = `"15"`

## 递归

### 例子说明

#### 阶乘

函数自己调用自己，比如用下面的函数计算5的阶乘

![image-20250107200545622](./Typora图片/image-20250107200545622.png)

调用自身，不断拆解问题，传递更小的参数，直到拆解成能直接得到答案的基准问题，得出结果后，逐层回归

![image-20250107200633635](./Typora图片/image-20250107200633635.png)

#### 斐波那契数列

![image-20250107201204590](./Typora图片/image-20250107201204590.png)

其问题拆解阶段就会被拆成一个树状图，问题越拆解越多

![image-20250107201308982](./Typora图片/image-20250107201308982.png)

这里会发现，很多相同的结果被重复计算，比如f(3)被计算了5次，f(4)被计算了三次，解决办法就是创造一个动态数组，把已经计算出来的结果存起来，这样就能把指数级的计算量优化成线性

#### 汉诺塔

![image-20250107205105386](./Typora图片/image-20250107205105386.png)

n等于1，可以直接把圆盘从from移动到to，移动完成直接完成直接return结束函数

![image-20250107205219522](./Typora图片/image-20250107205219522.png)

如果最大的圆盘要从from移动到to，前提是它上面没有其它圆盘

![image-20250107205424294](./Typora图片/image-20250107205424294.png)

那么其它圆盘只能在assist上，那就等同于将n-1个圆盘从from移动到assist,这个问题和当前编写函数解决的问题是一样的，只是规模从n变成n-1了，因此可以直接调用当前函数

![image-20250107205339404](./Typora图片/image-20250107205339404.png)

现在其他圆盘都移动到了assist，最大的圆盘可以直接从from到to，可以直接把这个步骤输出出来

![image-20250107205832266](./Typora图片/image-20250107205832266.png)

现在只需要把其他圆盘从assist移动到to，同样和当前编写函数要解决的问题相同，只是规模变成了n-1，因此又可以直接调用当前函数

![image-20250107205906789](./Typora图片/image-20250107205906789.png)

### 如何编写递归函数

确定问题，就是确定函数的参数

![image-20250107204655571](./Typora图片/image-20250107204655571.png)

当问题拆解到什么程度时，可以直接返回结果

![image-20250107204754957](./Typora图片/image-20250107204754957.png)

找到规模更小的子问题来解决当前的问题，就是把参数变得更接近基本问题

![image-20250107204845283](./Typora图片/image-20250107204845283.png)

## 实例

### 分形

![image-20250113204203356](./Typora图片/image-20250113204203356.png)

问题拆解

每一个大方块分成9块，但只需要其中5块，一直递归到尺寸为1的情况

![image-20250113202001198](./Typora图片/image-20250113202001198.png)

```C++
#include <iostream>
#include <math.h>
#include <vector>
#include <string>
using namespace std;
void draw(auto &matrix, int x, int y, int size) {
  if (size == 1) {
    matrix[x][y] = 'X';
    return;
  }
  int m1 = size / 3;
  int m2 = m1 * 2;
  draw(matrix, x, y, m1);
  draw(matrix, x, y + m2, m1);
  draw(matrix, x + m1, y + m1, m1);
  draw(matrix, x + m2, y, m1);
  draw(matrix, x + m2, y + m2, m1);
}
int main() {
  int n;
  while (cin >> n)
  {
    if (n == -1) break;
    int m = pow(3, n - 1);
    vector<string> matrix(m, string(m, ' '));
    draw(matrix, 0, 0, m);
    for (auto &line : matrix) {
      cout << line << endl;
    }
    cout << "-" << endl;
  }
}
```

## 匿名函数



![image-20250125145932056](./Typora图片/image-20250125145932056.png)

捕获

![image-20250125150347159](./Typora图片/image-20250125150347159.png)

初始化捕获

捕获时定义变量，起到不重复计算，又不污染外层作用域的作用，加上mutable就可以对初始化捕获所定义的变量进行修改

![image-20250125150456503](./Typora图片/image-20250125150456503.png)

匿名函数当作普通函数使用，只需要赋值给一个变量就行

![image-20250125150710221](./Typora图片/image-20250125150710221.png)

匿名函数的递归，只需要在参数中添加参数self代表自身，调用函数内部自己调用自己的时候，将函数自身将用self代表，传递自身给下一层递归时，也用self代表

![image-20250125150758664](./Typora图片/image-20250125150758664.png)

总结

![image-20250125145230145](./Typora图片/image-20250125145230145.png)

# 指针和内存

## 内存

8个连续的比特称之为字节，每个字节在内存中都有独立的地址，定义变量，就是在将某个地址开始的连续内存分配给变量，char两个字节，short四个字节，int4个字节等等

![image-20250126163440920](./Typora图片/image-20250126163440920.png)

![image-20250126163138510](./Typora图片/image-20250126163138510.png)

取出变量的内存地址，加上&地址符号

![image-20250126163553967](./Typora图片/image-20250126163553967.png)

## 指针

赋值指针

int* 称之为int指针，比如下面就是把存储i值的地址&i赋值给int指针ptr

![image-20250126164137686](./Typora图片/image-20250126164137686.png)
可以获取任意类型数据的地址，需要获取值的时候，转化成对应的类型来取值，用来当作函数参数或者用作接口非常好用

![image-20250126164334800](./Typora图片/image-20250126164334800.png)

获取指针所指向地址的值

加上*号就能根据指针的类型来解释地址对应的内存数据

改变原本的变量，指针的值也会变

![image-20250126164700429](./Typora图片/image-20250126164700429.png)

## 指针也叫解引用

- **获取地址**: 使用 `&` 操作符获取变量的地址，例如 `ptr = &var;`。
- **解引用**: 使用 `*` 操作符访问指针指向的值，例如 `int value = *ptr;`，就是用*去解开ptr这个地址，并引用这个内存地址存储的值

引用（Reference）

1. **定义**:

   - 引用是一个变量的别名，它另一个名字来引用同一个内存地址的变量。引用并不占用独立的内存空间。

2. **声明**:

   - 引用的声明以 `&` 符号开始。例如，`int& ref = var;` 声明了一个指向变量 `var` 的引用，用ref直接引用var内存地址上所存储的值。

   这里引用和指针所对应的源码是一样的

![image-20250127222712339](./Typora图片/image-20250127222712339.png)

定义空指针

![image-20250126165456034](./Typora图片/image-20250126165456034.png)

### 引用的常见用法

- `auto &score` 声明了一个引用 `score`，它可以直接访问 `scores` 向量中的每个元素。使用引用可以避免复制元素，提高效率。

```c++
vector<int> scores(m);
for (auto &score : scores) {
  cin >> score;
}
```



## 指针和复合类型

复合类型有数组，字符串，对象，函数

### 数组

将地址的第一值的地址赋值给指针，利用指针偏移，也就是移动地址的引用取出数组中所有的值

![image-20250127223759075](./Typora图片/image-20250127223759075.png)

直接将数组赋值给指针的时候，会进行隐式转换，会变成第一个元素的首地址

![image-20250127224048421](./Typora图片/image-20250127224048421.png)

![image-20250127224118442](./Typora图片/image-20250127224118442.png)

### 定义指向数组的指针

真正指向数据的指针如下图

![image-20250127224419059](./Typora图片/image-20250127224419059.png)

数组的类型是元素加长度，也就是int[5]

![image-20250127224245423](./Typora图片/image-20250127224245423.png)

因此可以如此定义

![image-20250127224822288](./Typora图片/image-20250127224822288.png)

使用using起别名进行定义

![image-20250127224930030](./Typora图片/image-20250127224930030.png)

### 数组作为参数传入函数

![image-20250127225417864](./Typora图片/image-20250127225417864.png)

### 二维数组

进行二维数组指针定义的时候，会隐式转换成  第一个长度为4的数组  的  指针

![image-20250128152831187](./Typora图片/image-20250128152831187.png)

如此情况可以把指针当做二维数组使用

![image-20250128153108915](./Typora图片/image-20250128153108915.png)

或者直接把二维数组看作是一维的，把第一个元素的地址赋值给指针，或者用第一个元素进行隐式转换，就可以通过地址偏移，线性的访问整个二维数组

![image-20250128153333696](./Typora图片/image-20250128153333696.png)

### 字符串

将带有终止符的**字符数组**首元素地址赋值给**字符指针**，就能把这个指针直接当字符串使用

![image-20250128154154045](./Typora图片/image-20250128154154045.png)

### 对象

对象指针解引用后，也可以跟原来一样通过点函数来使用

![image-20250128155235354](./Typora图片/image-20250128155235354.png)

### 函数

![image-20250128154643944](./Typora图片/image-20250128154643944.png)

![image-20250128154722939](./Typora图片/image-20250128154722939.png)

在前面添加返回值类型，后面顺序添加上参数的类型

![image-20250128154856650](./Typora图片/image-20250128154856650.png)

### 总结

![image-20250128155122061](./Typora图片/image-20250128155122061.png)

## C++指针内存模型

### 内存管理单元

内存管理单元会将物理内存切割成一页一页较小的内存，当有程序需要用到内存的时候，系统就会找到未被使用的内存，映射成一块连续的虚拟内存，这样程序就能申请到一块地址连续的区域，让内存得到充分的利用，且无法通过地址偏移访问其它的程序，只能操作分配的虚拟内存

![image-20250202203624704](./Typora图片/image-20250202203624704.png)

### 数据的生命周期

![image-20250202204203918](./Typora图片/image-20250202204203918.png)

静态：程序启动时分配，程序关闭时释放

动态：手动申请时分配，手动删除时释放

自动：在函数体内或花括号内定义时分配，函数结束或花括号结束时释放

![image-20250202205407108](./Typora图片/image-20250202205407108.png)

#### 自动生命周期

**内存的申请和释放完全自动**

一开始指针指向栈底部，定义变量的时候指针就向上移动对应类型所需的内存大小，偏移的部分就会分配给变量

<img src="C:\Typora图片\image-20250202205706512.png" alt="image-20250202205706512" style="zoom:33%;" /><img src="C:\Typora图片\image-20250202205746029.png" alt="image-20250202205746029" style="zoom:33%;" /><img src="./Typora图片/image-20250202205840459.png" alt="image-20250202205840459" style="zoom:33%;" />



进入花括号的时候，就会记录指针的当前地址

<img src="./Typora图片/image-20250202210002612.png" alt="image-20250202210002612" style="zoom:33%;" />

然后继续往上分配花括号定义的变量

![image-20250202210040697](./Typora图片/image-20250202210040697.png)

花括号结束的时候，指针回到原来记录的地方，花括号内分配的内存也会被释放

![image-20250202210155587](./Typora图片/image-20250202210155587.png)

当花括号结束的时候，内存虽然释放了，但是内存中的数据并没有被修改，后面定义y变量，根据栈后面输出栈的分配，y依然被分配到x的内存，输出仍然是123，当然不同编译器会进行优化

![image-20250205124826324](./Typora图片/image-20250205124826324.png)

![image-20250205124956520](./Typora图片/image-20250205124956520.png)

#### 静态生命周期

栈区大小只有8MB，如果定义的变量超过了栈区域，就要考虑别的内存了

![image-20250205135142641](./Typora图片/image-20250205135142641.png)

全局区有2G的大小，其中代码区的内容人无法操作，不考虑

![image-20250205135213580](./Typora图片/image-20250205135213580.png)

全局变量，在函数体外定义的变量，就算没有初始化，也有默认值

![image-20250205135649778](./Typora图片/image-20250205135649778.png)

静态变量就是在前面加上了static，其变量生命周期不由函数的结束决定，跟全局变量一样，但是无法在其它地方访问，可以既享受静态生命周期，又不会污染全局作用域

![image-20250205140308781](./Typora图片/image-20250205140308781.png)

#### 动态生命周期

其数据在堆区

当使用的空间比内存条还大的时候，mmu甚至会将硬盘也映射为虚拟内存，这里需要手动管理数据的生命周期，C++中用 `new` 来申请内存，并使用（）和  { }  来进行初始化，自身也有默认初始化

![image-20250205142024872](./Typora图片/image-20250205142024872.png)

在这里，new出来的数据在堆区，其中指针只是保存了堆区数据的地址，**这些地址信息被存在栈区，会收到自动生命周期的限制**

![image-20250205142721690](./Typora图片/image-20250205142721690.png)

当指针因为自动生命周期而消失，存在堆区的数据就没法访问了，把内存占用了但是其中数据没法用这就是**内存泄漏**，若是随着程序的运行，内存越来越少，运行速度也会越来越慢

![image-20250205142955018](./Typora图片/image-20250205142955018.png)

c++用`delete` 释放内存，传入指针就行

![image-20250205143227790](./Typora图片/image-20250205143227790.png)

指针指向的内存被释放后，指针的地址数据依然还在，但是之前释放的内存会被分配其它数据，在用原来的指针地址去访问，就会出现被分配到这块地址的其它数据，这就叫做**悬挂指针**

![image-20250205143345755](./Typora图片/image-20250205143345755.png)

所以释放完内存后，要将指针的值修改为 NULL或者

![image-20250205143647312](./Typora图片/image-20250205143647312.png)

当然指针之前传来传去，也会因为地址指向的内存被释放后，造成的悬挂指针问题，C++引入了**智能指针**，使用`make_shared`来初始化，申请的指针在栈中被销毁的时候，所申请的内存也会自动释放，如果指针复制了多次，当最后一个指针被销毁时，内存才会被释放

![image-20250208011857164](./Typora图片/image-20250208011857164.png)

智能指针`unique_ptr`无法被复制

![image-20250208013945063](./Typora图片/image-20250208013945063.png)

智能指针`weak_ptr`无法直接申请内存，但是可以等于shared_ptr和其它的weak_ptr

![image-20250208013815421](./Typora图片/image-20250208013815421.png)

![image-20250208014045186](./Typora图片/image-20250208014045186.png)

### 内存申请函数

`calloc`，先将内存中的数据清空后，返回首地址，返回值是void指针，申请内存大小为count * size 的空间

![image-20250208015001766](./Typora图片/image-20250208015001766.png)

`realloc`，对已分配的内存缩小或者扩大，返回值是void指针，参数是某个已分配内存的首地址和希望重新分配的大小

![image-20250208015916301](./Typora图片/image-20250208015916301.png)

![image-20250208015952192](./Typora图片/image-20250208015952192.png)

大小比原先内存小，直接释放差值部分

![image-20250208020030128](./Typora图片/image-20250208020030128.png)

比原先大就会向后扩容

![image-20250208020056480](./Typora图片/image-20250208020056480.png)

如果后面的内存已经被占用了，就会重新计算，找到一块完整的内存

![image-20250208020154656](./Typora图片/image-20250208020154656.png)

### 总结

![image-20250208014840745](./Typora图片/image-20250208014840745.png)

# 类

## 纯数据结构体

结构体的作用，就是将多个类型结合成一个整体，形成新的类型，比如名字和分数放在一起，然后进行统一操作

### 结构体的定义

![image-20250208125800251](./Typora图片/image-20250208125800251.png)

声明结构体类型的变量

直接在结构体后面跟上变量名，匿名结构体

声明单个变量，或者是数组都行

声明在栈区和堆区都行

![image-20250208125953204](./Typora图片/image-20250208125953204.png)

通过结构体变量加点号，就能够访问结构体内部的成员变量

![image-20250208130508773](./Typora图片/image-20250208130508773.png)

### 初始化

会最常见的就行

![image-20250208131228394](./Typora图片/image-20250208131228394.png)

直接赋值

![image-20250208131336901](./Typora图片/image-20250208131336901.png)

### 结构体嵌套

![image-20250208130923310](./Typora图片/image-20250208130923310.png)

### 结构体大小

首先以最大类型作为宽度构建矩阵

![image-20250208131840226](./Typora图片/image-20250208131840226.png)

按照定义顺序依次摆放

每个数据只能摆放着其大小倍数的位置，char就可以摆在任何地方，int只能摆在0或者4的位置，所以char后面空了三个字节

一行摆不下就要新起一行，所以short只能摆在第二行，double单独一行，最后的char新起一行

行*列，最终大小为32

![image-20250208131746076](./Typora图片/image-20250208131746076.png)

尽管根据内存对齐会极大浪费空间，但是可以加速计算机对数据的访问，但是可以从大到小定义数据节约内存

![image-20250208132354492](./Typora图片/image-20250208132354492.png)

对于嵌套的结构体，最大宽度以最大的数据类型宽度为准，对于数组，则是以元素类型作为宽度，不是整体宽度

![image-20250208132611231](./Typora图片/image-20250208132611231.png)

### 结构化绑定

把结构体内的变量绑定到新的变量上，但是必须绑定所有的变量，不能只绑定部分

这里让auto自动去推断数据类型

![image-20250208132818076](./Typora图片/image-20250208132818076.png)![image-20250208132846654](./Typora图片/image-20250208132846654.png)![image-20250208132854656](./Typora图片/image-20250208132854656.png)

这样可以很方便的变量结构体数据

![image-20250208133122253](./Typora图片/image-20250208133122253.png)

如果仅仅想使用某一个变量，可以用如下写法明晰语义

![image-20250208133157349](./Typora图片/image-20250208133157349.png)

### C++结构体新方式

使用pair对两个数据类型进行聚合

![image-20250208134500843](./Typora图片/image-20250208134500843.png)

超过两个元素，使用`tuple`

![image-20250208134805130](./Typora图片/image-20250208134805130.png)

使用tis函数，绑定成员变量到已定义的变量上

![image-20250209103010651](./Typora图片/image-20250209103010651.png)

把成员变量放入vector

![image-20250209103243814](./Typora图片/image-20250209103243814.png)

`push_back`相当于临时创建一个对象复制进入vector，`emplace_back`相当于直接在vector内部直接创建变量，少了一次复制的过程

`push_back`

![image-20250209103530536](./Typora图片/image-20250209103530536.png)

`emplace_back`

![image-20250209103444807](./Typora图片/image-20250209103444807.png)

### 总结

![image-20250209103820496](./Typora图片/image-20250209103820496.png)

## 实例

### 分数最高的学生名字

![image-20250209105558714](./Typora图片/image-20250209105558714.png)

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
int main() {
  int n;
  cin >> n;
  vector<pair<int, string>> vec(n);
  for (auto &[score, name] : vec) cin >> score >> name;
  cout << max_element(vec.begin(), vec.end())->second << endl;
}
```



### 结构体变量遍历

在第五人格巅峰七阶及以上的排位赛中，需要进行区域选择。我们将在本题中形式化、推广化的解决区域选择问题。

在地图中，共有 $n$ 台密码机，第 $i$ 台密码机的坐标为 $(x_i,y_i)$。在推广化的游戏中，有 $k$ 名求生者。每名求生者可以选择一台密码机作为其出生点，我们称被选择的密码机为 **出生密码机**。

监管者共有 $T$ 个出生点可供选择。第 $i$ 个可能的出生点坐标为 $(x_i,y_i)$。此时，由于“封禁”天赋的存在，离监管者最远的密码机将不能被破译。

**如果多台密码机与监管者的距离相同且最远，“封禁”天赋将会封禁这几台密码机中标号最小的那一台。**

请问在该 $T$ 个出生点中，有多少出生点，可以使某一台 **出生密码机** 被封禁。

请注意：坐标点 $(x_1,y_1)$ 与坐标点 $(x_2,y_2)$ 之间的距离为 $\sqrt{(x_1-x_2)^2+(y_1-y_2)^2}$。

**输入格式**

输入共 $n+T+k+1$ 行。

输入的第一行为三个整数 $n,k,T$。

接下来 $n$ 行，每行两个整数 $x_i,y_i$，表示一台密码机的坐标。

接下来 $k$ 行，每行两个整数 $x_i,y_i$，表示一名求生者选择出生密码机的坐标，保证该坐标在上面出现过。

接下来 $T$ 行，每行两个整数 $x_i,y_i$，表示监管者的一个出生点。

**输出格式**

输出一行一个整数，为答案。

**输入 #1**

```
4 2 2
-1 0
0 -1
2 0
0 2
-1 0
0 2
3 0
0 0
```

**输出 #1**

```
1
```

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
  int n, k, T;
  cin >> n >> k >> T;
  vector<tuple<int, int, bool>> points(n);
  for (auto &[x, y, _] : points) cin >> x >> y;
  // 用于出生点确认 的 结构体变量给定值
  while (k--)
  {
    int sx, sy;
    cin >> sx >> sy;
    for (auto &[x, y, withS] : points) {
      if (x == sx && y == sy) {
        withS = true;
        break;
      }
    }
  }
  // 判断那个监管者出生点能封禁求生者出生点
  int cnt = 0;
  while (T--) {
    int fx, fy;
    cin >> fx >> fy;
    int maxDist = 0;
    bool withSurvivor = false;
    for (auto &[x, y, withS] : points) {
      int dist = (x - fx) * (x - fx) + (y - fy) * (y - fy);
      if (dist > maxDist) {
        maxDist = dist;
        withSurvivor = withS;
      }
    }
    cnt += withSurvivor;
  }
  cout << cnt << endl;
}
```

### 按照标准差和名称排序

![image-20250217233046212](./Typora图片/image-20250217233046212.png)

```c++
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
int main() {
  int n, m;
  cin >> n >> m;
  using pds = pair<double, string>;
  vector<pds> vec(n);
  vector<int> scores(m);
  for (auto &[deviation, name] : vec) {
    cin >> name;
    double sum = 0;
    for (auto &score : scores) {
      cin >> score;
      sum += score;
    }
    double ave = sum / m;
    for (auto &score : scores) {
      deviation += (score - ave) * (score - ave);
    }
  }
  sort(vec.begin(), vec.end(), [](pds &a, pds &b) {
    if (a.first == b.first) return a.second < b.second;
    return a.first > b.first;
  });
  for (int i = 0; i < min((size_t)20, vec.size()); i++) {
    cout << vec[i].second << endl;
  }
}
```

## 类的封装性和成员函数

在C++中，结构体和类在能力上是完全等价的

### 结构体的控制访问

![image-20250218213441213](./Typora图片/image-20250218213441213.png)

### 类的封装性

除了开头的类型一个是struct另一个是class不同之外，类的内部成员变量的访问默认是private

![image-20250218213746556](./Typora图片/image-20250218213746556.png)

如果把类变量前加上public，那就和什么都不加的结构体在功能上没有区别

![image-20250218213905507](./Typora图片/image-20250218213905507.png)

### 类到对象

![image-20250218220907017](./Typora图片/image-20250218220907017.png)

### 构造函数

封装的目的是为了隐藏复杂逻辑，暴露有限的部分方便别人修改使用，如果所有内容都暴露出去，他人反而不好修改

那么既然类的内部成员都是被封装起来，无法修改，正常的数据初始化也无法完成，这个时候就要对外暴露相应的函数来进行初始化，用于初始化的函数叫做**构造函数**

![image-20250220214141104](./Typora图片/image-20250220214141104.png)

括号和花括号之间用冒号隔开进行成员变量初始化，如下这种格式写，就算参数名和成员变量名相同，也可以正常解析

![image-20250220214344228](./Typora图片/image-20250220214344228.png)

### 对象初始化

创建对象时，解析器会自动根据初始化对象的参数数量匹配正确的构造函数

比如这里 BigInt是类名，b就是对象

![image-20250228205838453](./Typora图片/image-20250228205838453.png)

### 析构函数

调用时机是栈中的对象被销毁，还是new出来的对象被手动delete，都会调用析构函数，就可以使用析构函数手动删除之前new出来的digits内存,~ 加上 构造函数名

![image-20250228210902330](./Typora图片/image-20250228210902330.png)

### 使用对象进行计算

通过在类的定义内写成员函数，这里成员函数返回的是对象ret，所以X在使用add()这个成员函数时，需要新定义一个对象Z来承接

![image-20250228212036412](./Typora图片/image-20250228212036412.png)

另一种定义成员函数的方式，可以直接累加到调用的对象上

![image-20250301200355242](./Typora图片/image-20250301200355242.png)

一个成员函数返回 `BigInt&` 时，它不创建对象的副本，而是返回当前对象的引用

把没有返回值的成员函数，可以将其自身返回，实现链式调用，`self_add`后可以再次调用`self_add`

![image-20250302224815758](./Typora图片/image-20250302224815758.png)

本质上是C++帮其传递了自身的指针，指针名字叫做this

![image-20250302231429057](./Typora图片/image-20250302231429057.png)

链式调用本质上还是：自己调用函数的同时，将自身作为参数传入，同时返会一个“自身”，接着在将刚才返还

![image-20250302225527561](./Typora图片/image-20250302225527561.png)

### 运算符重载

运算符重载可以把函数名改成对应的运算符

如何让创建的一个大数类像普通函数一样操作

![image-20250302231647910](./Typora图片/image-20250302231647910.png)

operator关键字

比如下面：通过重载 `operator+`，您可以定义如何使用 `+` 运算符对自定义类型（如类或结构体）进行操作。

<u>**下面构造函数中 `{ }` 里的内容进行了省略**</u>

![image-20250305182249708](./Typora图片/image-20250305182249708.png)

![image-20250302233032166](./Typora图片/image-20250302233032166.png)

可以进行重载的运算符

![image-20250302233256662](./Typora图片/image-20250302233256662.png)

比如cout<< ... <<endl就是对位运算符的重载计算

![image-20250302233451482](./Typora图片/image-20250302233451482.png)

一般运算符都有左右两个部分组成，表示对左边或者右边进行计算

![image-20250305180636876](./Typora图片/image-20250305180636876.png)

#### 一元运算符

重载一元运算符使用右边操作数

```
auto operator-(const Type &rhs){}
```

但是一元运算符都是改变自身，所以不需要传递参数都行，上边“-”号传递了右边的参数代表“减”，下边的“-”没有带参数表示“负”

比如下面：自增自减是返回自身，逻辑非是返回布尔，其它则是返回一个新对象

![image-20250305182619103](./Typora图片/image-20250305182619103.png)

#### 关系运算符

重载关系运算符

![image-20250305183029554](./Typora图片/image-20250305183029554.png)

可以只实现一个三路运算符，这样除了等于之外的其它运算符都会被自动实现

![image-20250306113124415](./Typora图片/image-20250306113124415.png)

#### 赋值运算符

其实就算不实现，同类型的对象间也可以直接赋值，但仅仅是对成员变量值的拷贝，

![image-20250312215904420](./Typora图片/image-20250312215904420.png)

#### 下标运算符

通过重载 `[]` 运算符来实现类似数组的访问方式：

```c++
class MyArray {
public:
    int& operator[](int index) {
        return data[index];  // 返回索引对应的元素
    }

private:
    int data[10] = {0};
};

MyArray arr;
arr[2] = 5;  // 使用下标访问符修改元素
int x = arr[2];  // 使用下标访问符读取元素
```

#### 函数调用运算符

重载了函数调用的圆括号后，就可以像函数一样使用对象和类，这种方式实现的类成为函数对象

```C++
class MyFunctor {
public:
    // 重载函数调用运算符
    int operator()(int x, int y) {
        return x + y;
    }
};

MyFunctor adder;
int result = adder(3, 4);  // 调用 operator()，返回 7
```

