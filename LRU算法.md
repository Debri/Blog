## LRU实现算法
> 要求put()和get()的时间复杂度为O(1),就是为一个常数
### 要求
1. 函数get（key）如果键存在于缓存中，则获取键的值，否则返回-1。要求时间复杂度O（1）
1. function put（key，value）如果键不存在，请设置或插入该值。如果值未定义，则忽略该操作。当缓存达到其容量时，它应该在插入新项目之前使最近使用的最少的项目无效。如果有两个或更多个具有相同频率的键，则使随机项无效。 要求时间复杂度O（1）

### 思路
* 队列 - 应该使用队列（双向队列），其中最近使用的页面位于前面，而最近未使用的页面位于后面。这将允许在O（1）时间内删除最近最少未使用的项目。

* 双向链表 - 我们应该使用双向链表（而不是数组）实现队列，允许在O（1）时间内页面移位操作。（比如，当需要将一个页面移动到队列的前面时）

* HashMap - 我们应该将键值使用hsah函数定位到cache存储页面的位置。这将允许get在O（1）时间内进行操作
### 实现
* 每当get()一个页面，我们返回其值，并将该页面移动到双向队列的前面
* 每当put()一个页面，如果页面已经存在，我们更新其值并将该页面移动到队列的前面，否则我们在Queue前面的缓存中添加一个新页面。但是，如果我们的缓存已经达到其容量，我们将从cache中删除最近最少未使用的页面（即队列中尾部的页面）。

### 代码

```
import java.util.HashMap;
import java.util.Map;

/**
 * Created by Liuqi
 * Date: 2017/5/9.
 */
public class LRUCache {

    int capacity;// 容量
    Map<Integer, Node> map;
    Node dummyEnd;
    Node dummyHead;
    int size;//当前大小

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        map = new HashMap<Integer, Node>();
        dummyEnd = new Node(0, 0);
        dummyHead = new Node(0, 0);
        dummyEnd.next = dummyHead;
        dummyHead.pre = dummyEnd;
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) {
            return -1;
        } else {
            remove(node);
            putToHead(node);
            return node.val;
        }
    }

    public void put(int key, int value) {
        Node oldNode = map.get(key);
        if (oldNode == null) {
            ++size;
            Node newNode = new Node(key, value);
            map.put(key, newNode);
            putToHead(newNode);
            if (size > capacity) {
                // 从LRU移除
                // 要先取出nextNode, 不然map里remove的就是错误的点，即dummy.next.next。
                Node nextNode = dummyEnd.next;
                remove(nextNode);
                // 从map移除
                map.remove(nextNode.key);
                --size;
            }
        } else {
            // 改变值，先移除，再放入头部
            oldNode.val = value;
            remove(oldNode);
            putToHead(oldNode);
        }
    }

    // 加到头和前一个点的中间
    public void putToHead(Node node) {
        Node preNode = dummyHead.pre;
        preNode.next = node;
        node.pre = preNode;
        dummyHead.pre = node;
        node.next = dummyHead;
    }

    //从双向链表中移除节点
    public void remove(Node node) {
        node.next.pre = node.pre;
        node.pre.next = node.next;
        // node 如果从尾部移除，将不会指向任何点。
        node.pre = null;
        node.next = null;
    }

    class Node {
        int key, val;
        Node pre, next;

        public Node(int key, int val) {
            this.key = key;
            this.val = val;
        }
    }
}

```