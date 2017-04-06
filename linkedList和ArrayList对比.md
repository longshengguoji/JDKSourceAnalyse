## linkedList和ArrayList对比
linkedList和ArrayList都实现了List接口，但是他们的工作原理却不一样。linkedlist是双向链表，而arrayList是数组
* 主要区别<br/>
arrayList是基于索引的数据结构，对其进行搜索和读取数据的速度很快，时间复杂度为O（1）,但是对数据的增删开销很大，时间复杂度为O(n)<br/>
linkedList是基于双向链表的数据结构，对其进行增删的开销很小，时间复杂度为O(1)，但是对齐进行搜索和读取数据的开销很大，时间复杂度为O(n)<br/>
在存储时linkedList需要更多的内存，其存储时每个节点存储的是实际的数据和其前后节点的位置；而arrayList存储的是实际的数据
* 适用场景<br/>
适用arrayList的场景：应用需要频繁的访问数据，而对数据的增删操作较少<br/>
适用linkedList的场景：应用需要频繁的对数据进行增删操作，对数据的访问较少
