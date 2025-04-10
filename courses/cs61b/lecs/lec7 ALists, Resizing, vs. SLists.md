
使用链表访问元素很慢, 因此可以考虑将数组封装成 ALists 来获得快速的存取

ALists 的 api 跟 DLists 一样, 对用户隐藏实现细节

调整容量:
- 当数组太满时, 扩容为原来容量的两倍, 使用 `system.Arraycopy()` 进行数据迁移
- 数组太空时则缩容

泛型:
- java 中数组是不支持使用泛型声明的, 因此需要声明为 `Object`, 然后类型转换为该泛型
- `items = new Item[100] ==> items = (Item[]) new Object[100]`

