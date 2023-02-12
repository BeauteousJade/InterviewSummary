# Fresco

## 1. 结构
>1. DraweeView:View 层，继承于ImageView
>2. DraweeHierarchy：Model 层，用于维护相关的图片数据
>3. DraweeController：C层，主要是处理图片加载和处理的相关逻辑。
>4. DraweeHolder：View 和Mode、C的粘合剂。


###  2. DraweeController 的构造逻辑
Fresco的init给每一个View设置了一个静态sDraweecontrollerbuildersupplier，然后在DraweeView的init阶段拿到一个ControllerBuilder。在setUri的时候，会通过Builder创建一个DraweeController。

### 4. 通过 DataSource 发起图片加载
DraweeController的submitRequest方法发起请求，拿到一个DataSource对象进行请求。在创建DataSource 的时候，会指定创建Producer责任链，用来分布请求、解析和处理图片。