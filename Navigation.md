1. 概述
2. 基本使用
> 1.  准备工作--定义xml，graph文件，以及NavHostFragment 的使用
> 2.  几种跳转逻辑--action跳转、deepLink跳转，以及activity之前的跳转
> 3.  特别注意d循环跳转

3. 基本结构和类的解释
> 1. 基本结构
> 2. 核心类，NavHostFragment、NavHostController、Navigator、NavInflater、NavDestination及其子类。

4. graph文件解析
> 重点分析NavInflater的解析过程

5. 跳转逻辑的实现
>1. 初始化的跳转，即graph的startDestination
>2. action的跳转
>3. deeplink跳转
>4. popTo 和 popUpToInclusive的特别分析

6. FragmentNavigator的分析
> 1. 怎么跳转的
> 2. 怎么返回的，同时怎么维护返回栈的。popBackStack和navigatUp。

6. 如何自定义Navigator，如何自定义传参？
>1.  怎么保证生命周期正确的。
>2.  怎么版本返回栈是正确的。
>3. 传参的基本方式，标准传参和非标准传参。

7. Navigation的一些设计美学和缺点
> 美学：<br>
>1. graph文件的存在，使得跳转流程可视化、配置化<br>
>2. Navigator的独立，且支持自定义化。<br>
>3. 传参方法接口化，完美适配自定义需求<br>
>4. Destination的抽象化。<br>
>缺点：<br>
>1. NavOption的final，不支持自定义属性<br>
>2. NavInflater的final，不支持自定义。

