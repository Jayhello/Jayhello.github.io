定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。



## 1 思路

这题比较有意思：

1. 使用常规的数据结构实现栈的话，低层用一个数组或链表，那么查找最小值要遍历一次，时间复杂度为O（n）。

2. 若要求min()时间复杂度为O（1）的话，说明我们要记录下最小值是多少，在调用min()的时候快速返回这个值。这样就有一个思路：

   - 在push()的时候不断地刷新最小值min，时间复杂度为O(1)。
   - 在min()的时候直接返回min，时间复杂度为O(1)。
   - 在pop()的时候检查一下，若弹出的是min，则遍历一次找出新的min，若弹出的不是min，则无需遍历。时间复杂度为O(n)。

   空间复杂度为O(n)。

3. 上面的思路可以满足题目要求，只有pop()的时候可能会多废一点时间。另一种思路是在栈的每个元素旁边都记录一个min，这个min表示以这个元素为栈顶的时候，栈中的最小值。

   - 在push()的时候比较当前栈顶元素的min与当前元素的大小，取较小者为新的min。
   - 在min()的时候返回栈顶元素的min。
   - 在pop()的时候正常pop。

   以上三种操作的时间复杂度都为O(1)，但是整个栈多消耗了一倍的空间。与第二种思路相比孰优孰劣还需要根据实际情况取舍。



## 2 java代码

下面给出第3中思路的代码：

```java
import java.util.Stack;

public class Solution {
    // 构建这样一个栈：
    // 1.每个节点存放两个值，val为它本来的值，min是此节点及其下所有节点中的最小值
    // 2.在push的时候构建一个新节点，这个节点的min是当前栈顶的min和此节点的val中的较小者
    private class Element {
        public int val;
        public int min;
    }
    
    private Stack<Element> stack = new Stack<>();
    
    public void push(int node) {
        Element newElement = new Element();
        newElement.val = node;
        
        // 计算min
        // 若目前栈为空则设置为当前节点的值
        if (stack.empty()) {
            newElement.min = node;
        } else {
            // 否则取栈顶元素与当前元素中的较小者
            Element top = stack.peek();
            if (node < top.min) {
                newElement.min = node;
            } else {
                newElement.min = top.min;
            }
        }

        
        stack.push(newElement);
    }
    
    public void pop() {
        stack.pop();
    }
    
    public int top() {
        return stack.peek().val;
    }
    
    public int min() {
        return stack.peek().min;
    }
}
```

