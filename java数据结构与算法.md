字符匹配:KMP算法
汉诺塔:分治算法
八皇后问题:回溯算法
马踏棋盘算法:图的深度优化遍历算法(DFS)+贪心算法优化
约瑟夫问题(丢手帕问题)
修路问题=>最小生成树(普利姆算法)
最短路径问题=>弗洛伊德算法

## 007稀疏数组
用时间换取空间


### 单链表介绍和内存布局
1) 链表是以节点的形式来存储
2) 每个节点包含data域,next域:指向下一个节点。
3) 如图,发现链表的各个节点不一定是连续存储。
4) 链表分带头节点的链表和没有头节点的链表,根据实际的需求来确定。

```java
   public class test {
    public static void main(final String[] args) {
        // 进行测试
        // 先创建节点
        HeroNote hero1 = new HeroNote(1, "宋江", "及时雨");
        HeroNote hero2 = new HeroNote(2, "卢俊义", "玉麒麟");
        HeroNote hero3 = new HeroNote(3, "吴用", "智多星");
        HeroNote hero4 = new HeroNote(4, "林冲", "豹子头");

        SingleLinkedList singleLinkedList = new SingleLinkedList();
        singleLinkedList.add(hero1);
        singleLinkedList.add(hero2);
        singleLinkedList.add(hero3);
        singleLinkedList.add(hero4);
        
        singleLinkedList.list();
    }
}

/**
 * HeroNote
 */
class HeroNote {
    public int no;
    public String name;
    public String nickname;
    public HeroNote next;

    // 构造器
    public HeroNote(int no, String name, String nickname) {

    }
}

class SingleLinkedList {
    // 先初始化一个头节点,头节点不要动,不存放具体的数据
    private HeroNote head = new HeroNote(0, "", "");

    public void add(HeroNote heroNote) {
        HeroNote temp = head;
        while (true) {
            if (temp.next == null) {
                break;

            }
            temp = temp.next;
        }
        temp.next = heroNote;
    }

    public void list() {
        if (head.next == null) {
            System.out.println("链表为空");
            return;
        }
        HeroNote temp = head.next;
        while (true) {
            if (temp == null) {
                break;
            }

        }
        System.out.println(temp);
        temp = temp.next;
    }
}
```