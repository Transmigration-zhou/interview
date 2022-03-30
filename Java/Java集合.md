![img](https://gitee.com/Transmigration_zhou/pic/raw/master/img/20220330152143.png)

## List

### ArrayList





### HashMap和Hashtable的区别

HashMap是Hashtable的轻量级实现

|                     HashMap                     |                      Hashtable                       |
| :---------------------------------------------: | :--------------------------------------------------: |
|                   线程不安全                    |                       线程安全                       |
|               允许有null的键和值                |                 不允许有null的键和值                 |
|                   效率高一点                    |                       效率稍低                       |
|    方法不是Synchronize(同步)的，要提供外同步    |                 方法是Synchronize的                  |
|        有containsvalue和containsKey方法         |                    有contains方法                    |
| 迭代器采用的是Iterator，是快速失败（Fail-Fast） | 迭代器**还**采用Enumeration，是安全失败（Fail-Safe） |

> **快速失败 Fail-Fast**:一旦发生异常，立即停止当前的操作，并上报给上层的系统来处理这些故障。
>
> **安全失败 Fail-Safe**:在故障发生之后会维持系统继续运行。

**初始化容量不同**：HashMap 的初始容量为 16，Hashtable 初始容量为 11。两者的负载因子默认都是 0.75；

**扩容机制不同**：当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1；

