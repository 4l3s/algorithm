# algorithm
## 题目一 基于栈的虚拟机
【软件认证】基于栈的虚拟机
请你实现一个基于栈(先进后出)的虚拟机，执行输入的指令cmds后返回从栈底到栈顶的数据。
cmds是16进制字符串表示的字节流，字符为[0-9A-F]，字母为大写，每两个字符表示1个字节。内容是一串紧密排列的变长指令，每条指令格式为 [Op][Body]：
Op 为指令码，固定为2个字节；
Body 为操作内容，长度和含义由 Op 决定。
| 指令 | Op (hex) | Body 长度 | 含义 |
|------|--------|----------|------|
| PUSH | `C001` | 4 字节（8 hex 字符） | 压入 Body 的值 |
| POP  | `C002` | 0 字节 | 弹出（忽略空栈） |
| ADD  | `C011` | 4 字节（8 hex 字符） | A = pop() or 0; push(A + B) |
| SUB  | `C012` | 4 字节（8 hex 字符） | A = pop() or 0; push(A - B) |
| JGT  | `C021` | 2 字节（4 hex 字符） | A=pop() or 0, B=pop() or 0; if A >= B → 跳转到 字节偏移 = Body 值 |

这到提有点意思，其他几个操作都没啥，关键是这个跳转，你要跳转到合适的字节去，我当时想法就是遍历变cmds字符串，然后找到每一个op的位置字节号是多少保存起来，对应的字符串index也保存起来。其实在这里应该能找到规律了，
更好的方法是，不用遍历  cmds 是 hex 字符串，2 字符 = 1 字节 → 所以 字节偏移 × 2 = 字符索引。直接就定位，后面按照字节循环这个比较好
import java.util.*;

public class StackVM {
    public static Long[] execute(String cmds) {
        Deque<Long> stack = new ArrayDeque<>();
        int pc = 0; // program counter: 当前字节偏移（不是字符索引！）

        while (pc * 2 < cmds.length()) {
            // 读取 Op: 2 字节 = 4 hex 字符
            int opStart = pc * 2;
            if (opStart + 4 > cmds.length()) break;
            String opHex = cmds.substring(opStart, opStart + 4);

            switch (opHex) {
                case "C001": // PUSH
                    if (opStart + 4 + 8 > cmds.length()) return toArray(stack);
                    String bodyHex = cmds.substring(opStart + 4, opStart + 12); // 4字节=8字符
                    long value = Long.parseLong(bodyHex, 16);
                    stack.push(value);
                    pc += (2 + 4); // 2(Op) + 4(Body) 字节
                    break;

                case "C002": // POP
                    if (!stack.isEmpty()) {
                        stack.pop();
                    }
                    pc += 2; // 仅 Op
                    break;

                case "C011": // ADD
                    if (opStart + 4 + 8 > cmds.length()) return toArray(stack);
                    bodyHex = cmds.substring(opStart + 4, opStart + 12);
                    long B = Long.parseLong(bodyHex, 16);
                    long A = stack.isEmpty() ? 0L : stack.pop();
                    stack.push(A + B);
                    pc += (2 + 4);
                    break;

                case "C012": // SUB
                    if (opStart + 4 + 8 > cmds.length()) return toArray(stack);
                    bodyHex = cmds.substring(opStart + 4, opStart + 12);
                    B = Long.parseLong(bodyHex, 16);
                    A = stack.isEmpty() ? 0L : stack.pop();
                    stack.push(A - B);
                    pc += (2 + 4);
                    break;

                case "C021": // JGT
                    if (opStart + 4 + 4 > cmds.length()) return toArray(stack);
                    String offsetHex = cmds.substring(opStart + 4, opStart + 8); // 2字节=4字符
                    long offsetBytes = Long.parseLong(offsetHex, 16);

                    long A = stack.isEmpty() ? 0L : stack.pop();
                    long B = stack.isEmpty() ? 0L : stack.pop();

                    if (A >= B) {
                        pc = (int) offsetBytes; // 跳转到字节偏移位置
                    } else {
                        pc += (2 + 2); // 2(Op) + 2(Body)
                    }
                    break;

                default:
                    // 未知指令，跳过 2 字节（保守处理）
                    pc += 2;
                    break;
            }
        }

        return toArray(stack);
    }

    // 从栈底到栈顶转为 Long[]
    private static Long[] toArray(Deque<Long> stack) {
        return stack.toArray(new Long[0]);
    }
}





2这道算法也很有意思,刚开始的的描述很有误导性，以为要用优先队列deque，其实暴力使用才是最好的
【软件认证】选择最匹配内存部署虚拟机

虚拟化是云计算的重要技术之一，使用虚拟化技术，可以在一台物理机上部署一个或多个虚拟机，以优化资源的使用。

现有一批物理机记录于数组 capacities中，capacities[i] 表示编号为 i 的物理机初始内存大小；同时给出一批虚拟机部署请求requests， requests[j] 表示某虚拟机的所需内存。

请按如下规则依次处理每个虚拟机部署请求，并返回每个虚拟机部署所在的物理机编号（或者 -1）：

如果所有物理机的可用内存都不足，则该虚拟机部署失败，结果为 -1；
否则，在满足虚拟机所需内存的所有物理机中，选择可用内存最小的；若仍有多台，则选择其中编号最小的。

    static int[] dispatchRequestsOpti1(int[] capacities, int[] requests) {

       int[] valid=capacities.clone();
       int minVal=Integer.MAX_VALUE;
       int bestIndex=-1;
       List<Integer> ans= new ArrayList<>();
        for (int i = 0; i < requests.length; i++) {
            minVal=Integer.MAX_VALUE;
            bestIndex=-1;
            for (int j = 0; j < valid.length;  j++) {
                if(requests[i]<=valid[j]){
                    if(valid[j]<minVal){
                        minVal=valid[j];
                        bestIndex=j;
                    }
                }
            }
            if(bestIndex != -1){
                valid[bestIndex]-=requests[i];
                ans.add(bestIndex);
            }else{
                ans.add(bestIndex);
            }
        }

        return ans.stream().mapToInt(Integer::intValue).toArray();
    }
不用队列的原因是：体现在：必须用一个临时容器（如 List）把所有 poll 出来的 valid 但不满足的元素存起来，处理完当前请求后再全部 offer 回堆中。
但正因为这个操作太笨重，实际工程和面试中都推荐直接用暴力扫描——它天然规避了这个问题，且简单可靠。

这里还可以优化。暴力搜索复杂度是O(m*n),在java中5千万一般需要一秒，现代cpu是10的8次方 ops/sec
使用 TreeMap（或优先队列 + 懒删除 + 正确策略）
我们可以维护一个结构：
TreeMap<Integer, TreeSet<Integer>> memToMachines
key: 可用内存大小
value: 所有具有该内存的物理机编号（用 TreeSet 自动排序）
这样：
查找 ≥ req 的最小内存：map.ceilingKey(req)
获取该内存下编号最小的机器：treeSet.first()
更新机器内存：从旧内存集合中移除，加入新内存集合
所有操作 O(log n)，总复杂度 O(m log n)

import java.util.*;

public class Solution {
    public int[] deployVMs(int[] capacities, int[] requests) {
        int n = capacities.length;
        int[] available = Arrays.copyOf(capacities, n);
        
        // TreeMap: 可用内存 -> 有序的机器编号集合
        TreeMap<Integer, TreeSet<Integer>> memToMachines = new TreeMap<>();
        
        // 初始化
        for (int i = 0; i < n; i++) {
            memToMachines.computeIfAbsent(capacities[i], k -> new TreeSet<>()).add(i);
        }
        
        int[] result = new int[requests.length];
        
        for (int i = 0; i < requests.length; i++) {
            int req = requests[i];
            
            // 找到 >= req 的最小可用内存
            Integer mem = memToMachines.ceilingKey(req);
            if (mem == null) {
                result[i] = -1;
                continue;
            }
            
            // 获取该内存下编号最小的机器
            TreeSet<Integer> machines = memToMachines.get(mem);
            int machineId = machines.first();
            
            // 从旧内存集合中移除
            machines.remove(machineId);
            if (machines.isEmpty()) {
                memToMachines.remove(mem);
            }
            
            // 更新机器内存
            int newMem = mem - req;
            available[machineId] = newMem;
            
            // 加入新内存集合
            if (newMem > 0) { // 可选：0 内存也可以保留
                memToMachines.computeIfAbsent(newMem, k -> new TreeSet<>()).add(machineId);
            }
            
            result[i] = machineId;
        }
        
        return result;
    }
}
