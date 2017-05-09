## LRUʵ���㷨
> Ҫ��put()��get()��ʱ�临�Ӷ�ΪO(1),����Ϊһ������
### Ҫ��
1. ����get��key������������ڻ����У����ȡ����ֵ�����򷵻�-1��Ҫ��ʱ�临�Ӷ�O��1��
1. function put��key��value������������ڣ������û�����ֵ�����ֵδ���壬����Ըò�����������ﵽ������ʱ����Ӧ���ڲ�������Ŀ֮ǰʹ���ʹ�õ����ٵ���Ŀ��Ч�����������������������ͬƵ�ʵļ�����ʹ�������Ч�� Ҫ��ʱ�临�Ӷ�O��1��

### ˼·
* ���� - Ӧ��ʹ�ö��У�˫����У����������ʹ�õ�ҳ��λ��ǰ�棬�����δʹ�õ�ҳ��λ�ں��档�⽫������O��1��ʱ����ɾ���������δʹ�õ���Ŀ��

* ˫������ - ����Ӧ��ʹ��˫���������������飩ʵ�ֶ��У�������O��1��ʱ����ҳ����λ�����������磬����Ҫ��һ��ҳ���ƶ������е�ǰ��ʱ��

* HashMap - ����Ӧ�ý���ֵʹ��hsah������λ��cache�洢ҳ���λ�á��⽫����get��O��1��ʱ���ڽ��в���
### ʵ��
* ÿ��get()һ��ҳ�棬���Ƿ�����ֵ��������ҳ���ƶ���˫����е�ǰ��
* ÿ��put()һ��ҳ�棬���ҳ���Ѿ����ڣ����Ǹ�����ֵ������ҳ���ƶ������е�ǰ�棬����������Queueǰ��Ļ��������һ����ҳ�档���ǣ�������ǵĻ����Ѿ��ﵽ�����������ǽ���cache��ɾ���������δʹ�õ�ҳ�棨��������β����ҳ�棩��

### ����

```
import java.util.HashMap;
import java.util.Map;

/**
 * Created by Liuqi
 * Date: 2017/5/9.
 */
public class LRUCache {

    int capacity;// ����
    Map<Integer, Node> map;
    Node dummyEnd;
    Node dummyHead;
    int size;//��ǰ��С

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
                // ��LRU�Ƴ�
                // Ҫ��ȡ��nextNode, ��Ȼmap��remove�ľ��Ǵ���ĵ㣬��dummy.next.next��
                Node nextNode = dummyEnd.next;
                remove(nextNode);
                // ��map�Ƴ�
                map.remove(nextNode.key);
                --size;
            }
        } else {
            // �ı�ֵ�����Ƴ����ٷ���ͷ��
            oldNode.val = value;
            remove(oldNode);
            putToHead(oldNode);
        }
    }

    // �ӵ�ͷ��ǰһ������м�
    public void putToHead(Node node) {
        Node preNode = dummyHead.pre;
        preNode.next = node;
        node.pre = preNode;
        dummyHead.pre = node;
        node.next = dummyHead;
    }

    //��˫���������Ƴ��ڵ�
    public void remove(Node node) {
        node.next.pre = node.pre;
        node.pre.next = node.next;
        // node �����β���Ƴ���������ָ���κε㡣
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