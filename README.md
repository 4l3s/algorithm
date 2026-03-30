# algorithm
## 题目1：基于栈的虚拟机
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
```java
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
```




## 题目2：选择最匹配内存部署虚拟机
这道算法也很有意思,刚开始的的描述很有误导性，以为要用优先队列deque，其实暴力使用也可以，你以为暴力可以其实用红黑数才是完美的
【软件认证】选择最匹配内存部署虚拟机

虚拟化是云计算的重要技术之一，使用虚拟化技术，可以在一台物理机上部署一个或多个虚拟机，以优化资源的使用。

现有一批物理机记录于数组 capacities中，capacities[i] 表示编号为 i 的物理机初始内存大小；同时给出一批虚拟机部署请求requests， requests[j] 表示某虚拟机的所需内存。

请按如下规则依次处理每个虚拟机部署请求，并返回每个虚拟机部署所在的物理机编号（或者 -1）：

如果所有物理机的可用内存都不足，则该虚拟机部署失败，结果为 -1；
否则，在满足虚拟机所需内存的所有物理机中，选择可用内存最小的；若仍有多台，则选择其中编号最小的。
```java
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
```
**不用队列的原因是：**体现在：必须用一个临时容器（如 List）把所有 poll 出来的 valid 但不满足的元素存起来，处理完当前请求后再全部 offer 回堆中。
但正因为这个操作太笨重，实际工程和面试中都推荐直接用暴力扫描——它天然规避了这个问题，且简单可靠。

**更完美的方法(红黑树)：**
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
```java
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
```
这里面有个在 Java 标准库中，只有基于红黑树（Red-Black Tree）实现的有序集合/映射才提供 ceiling、floor 等导航方法。这些方法是 NavigableMap 和 NavigableSet 接口定义的核心功能。
| 方法 | 含义 |
|------|------|
| `lowerKey(key)`   | < key 的最大值（严格小于）|
| `floorKey(key)`   | ≤ key 的最大值 |
| `ceilingKey(key)` | ≥ key 的最小值 |
| `higherKey(key)`  | > key 的最小值（严格大于）|
在熟悉一下 如何定义比较器 TreeSet<MyClass> set = new TreeSet<>(Comparator.comparing(MyClass::getName));

## 题目3 简易画图
请设计一个简易画图程序，画布由 100 * 100 个大小相同的方格组成，左上角方格的位置为 [0, 0]

屏幕中一个矩形位置（左上角）用 [row, col] 表示，宽和高分别用 width 和 height 表示。

例如：黄色矩形的行、列位置为 [2, 7]，宽、高分别为 1、3

PbrushSystem() — 初始化系统
drawRectangle(int row, int col, int width, int height) -- 在位置 [row, col] 绘制一个大小为 width * height 的矩形：若出现重叠或超出边界，则直接返回 false；否则，绘制成功并返回 true。
eraseArea(int row, int col, int width, int height) -- 选中所有与区域 [row, col, width, height] 有重叠的矩形完整擦除，返回擦除的矩形个数。
queryArea() -- 计算完全覆盖所有矩形的最小矩形区域的面积，若无矩形则为 0 

思考：
这里有个意思的判断，如何判断相交：
之前想到是任何一个点在一个矩形里面就相交，比如矩形B的四个点中任何一个点在A中就行，但其实这个考虑不周，还有一种情况是十字相交的，
所以 **重叠检查：使用标准矩形重叠算法（两个矩形不重叠的条件是：一个在另一个的左边、右边、上边或下边）**
```java
public class PbrushSystem {
    // 画布大小
    private static final int CANVAS_SIZE = 100;
    
    // 存储所有绘制的矩形
    private List<Rectangle> rectangles;
    
    // 定义矩形类
    private static class Rectangle {
        int row, col, width, height;
        
        public Rectangle(int row, int col, int width, int height) {
            this.row = row;
            this.col = col;
            this.width = width;
            this.height = height;
        }
        
        // 检查是否与另一个矩形重叠
        public boolean overlapsWith(Rectangle other) {
            return !(this.col + this.width <= other.col || 
                    other.col + other.width <= this.col ||
                    this.row + this.height <= other.row || 
                    other.row + other.height <= this.row);
        }
        
        // 检查是否在画布边界内
        public boolean isWithinBounds() {
            return row >= 0 && col >= 0 && 
                   row + height <= CANVAS_SIZE && 
                   col + width <= CANVAS_SIZE;
        }
    }
    
    public PbrushSystem() {
        rectangles = new ArrayList<>();
    }
    
    public boolean drawRectangle(int row, int col, int width, int height) {
        // 检查参数有效性
        if (width <= 0 || height <= 0) {
            return false;
        }
        
        Rectangle newRect = new Rectangle(row, col, width, height);
        
        // 检查是否超出边界
        if (!newRect.isWithinBounds()) {
            return false;
        }
        
        // 检查是否与现有矩形重叠
        for (Rectangle existing : rectangles) {
            if (newRect.overlapsWith(existing)) {
                return false;
            }
        }
        
        // 绘制成功，添加到列表
        rectangles.add(newRect);
        return true;
    }
    
    public int eraseArea(int row, int col, int width, int height) {
        if (width <= 0 || height <= 0) {
            return 0;
        }
        
        Rectangle eraseRegion = new Rectangle(row, col, width, height);
        
        // 找出所有与擦除区域重叠的矩形
        Iterator<Rectangle> iterator = rectangles.iterator();
        int erasedCount = 0;
        
        while (iterator.hasNext()) {
            Rectangle rect = iterator.next();
            if (rect.overlapsWith(eraseRegion)) {
                iterator.remove();
                erasedCount++;
            }
        }
        
        return erasedCount;
    }
    
    public int queryArea() {
        if (rectangles.isEmpty()) {
            return 0;
        }
        
        // 初始化边界
        int minRow = Integer.MAX_VALUE;
        int minCol = Integer.MAX_VALUE;
        int maxRow = Integer.MIN_VALUE; // 最大行坐标（底部）
        int maxCol = Integer.MIN_VALUE; // 最大列坐标（右边界）
        
        // 遍历所有矩形，找到整体的边界
        for (Rectangle rect : rectangles) {
            minRow = Math.min(minRow, rect.row);
            minCol = Math.min(minCol, rect.col);
            maxRow = Math.max(maxRow, rect.row + rect.height);
            maxCol = Math.max(maxCol, rect.col + rect.width);
        }
        
        // 计算最小包围矩形的面积
        int totalWidth = maxCol - minCol;
        int totalHeight = maxRow - minRow;
        
        return totalWidth * totalHeight;
    }
}
```


## 题目四 计算任务运行总耗时
**(1)带依赖的任务调度问题**
 带资源约束的依赖任务调度问题

现有一批待执行的任务tasks，tasks[i]=[taskType，depend]，i为任务编号，taskType 表示任
务类型，共有 6 种，不同类型的任务执行耗时参考下面的表格，depend 表示所依赖的任务编号（-1表
示无依赖）。
请设计一个满足如下要求的调度模块，完成上述任务的执行，返回总耗时：
·系统中固定有6个NPU在并行运行，分别执行这6 种类型的任务（一个NPU对应一种类型）；一个
NPU内的任务串行执行。
·对于无依赖的任务，都是ready状态的；对于存在依赖的任务，须等待所依赖的任务执行完才能变成
ready状态。
当NPU空闲时，选择处于ready状态的任务立即执行，若有多个，选择编号小的。
6 种任务类型对应的执行时间
```java
taskMap.put(1, 9);
taskMap.put(2, 15);
taskMap.put(3, 21);
taskMap.put(4, 18);
taskMap.put(5, 30);
taskMap.put(6, 42);
```
 解题思路
解析输入：构建任务列表 tasks[i] = [type, depend]
初始化：
finishTime[i]：任务 i 完成时间（初始 -1）
npuFree[7]：NPU 1~6 的空闲时间（初始为 0）
readyQueue：优先队列（按任务编号升序）
初始 ready 任务：depend == -1 的任务加入 readyQueue
模拟调度：
每次从 readyQueue 取出编号最小的任务
获取其类型 t，查表得执行时间 duration
该任务开始时间 = max(npuFree[t], 当前全局时间?) → 实际上我们不需要全局时间，只需按事件驱动
关键：任务开始时间 = npuFree[t]（因为 NPU 必须等自己空闲）
任务结束时间 = npuFree[t] + duration
更新 npuFree[t] = 结束时间
更新 finishTime[i] = 结束时间
检查所有未完成任务，若其依赖的任务已完成，则变为 ready（加入队列）
重复直到所有任务完成
返回最大 finishTime
⚠️ 注意：不能用“全局时间”推进，而是事件驱动——每次执行一个 ready 任务，然后检查新 ready 任务。
但有一个陷阱：多个任务可能在不同时间点 become ready，但我们只在任务完成时才检查依赖。
✅ 正确做法：每当一个任务完成，就遍历所有未完成任务，看是否有依赖已满足，若有且未入队，则加入 readyQueue。
由于 readyQueue 是优先队列（按编号），每次取最小编号即可。
```java
    int schedule(List<Task> tasks) {
        if (tasks == null || tasks.isEmpty()) {
            return 0;
        }

        int n = tasks.size();
        int[] finishTime = new int[n]; // finishTime[i] = 任务 i 的完成时刻
        Arrays.fill(finishTime, -1);

        // npuFree[type] = 类型为 type 的 NPU 下次空闲时间（type 1～6）
        long[] npuFree = new long[7]; // 索引 0 不用，1～6 对应类型

        // ready 队列：存储任务编号（即 List 中的索引），按编号升序
        PriorityQueue<Integer> readyQueue = new PriorityQueue<>();

        // 初始化：将无依赖的任务加入 ready 队列
        for (int i = 0; i < n; i++) {
            if (tasks.get(i).depend == -1) {
                readyQueue.offer(i);
            }
        }

        // 调度主循环
        while (!readyQueue.isEmpty()) {
            int taskId = readyQueue.poll(); // 取编号最小的 ready 任务
            Task task = tasks.get(taskId);
            int type = task.taskType;
            int dep = task.depend;

            // 获取该类型任务的执行时间
            int duration = TASK_DURATION.getOrDefault(type, 0);
            if (duration <= 0) {
                throw new IllegalArgumentException("Unknown task type: " + type);
            }

            // 计算开始时间：必须等 NPU 空闲 AND 依赖任务完成
            long startTime;
            if (dep == -1) {
                startTime = npuFree[type];
            } else {
                // 依赖任务已完成，其完成时间已知
                startTime = Math.max(npuFree[type], finishTime[dep]);
            }

            long endTime = startTime + duration;
            npuFree[type] = endTime;
            finishTime[taskId] = (int) endTime;

            // 激活所有直接依赖此任务的后续任务
            for (int j = 0; j < n; j++) {
                if (finishTime[j] == -1 && tasks.get(j).depend == taskId) {
                    readyQueue.offer(j);
                }
            }
        }

        // 总耗时 = 所有任务完成时间的最大值
        return Arrays.stream(finishTime).max().orElse(0);
}
```






1. 必须或通常使用优先队列的场景
当调度策略需要根据任务的动态属性（如优先级、截止时间、剩余运行时间等）来决定下一个执行哪个任务时，优先队列是最高效的选择（通常基于堆实现，插入和取出最优元素的时间复杂度为 
O(log n)
O(logn) ）。
优先级调度 (Priority Scheduling): 显然需要，总是选择优先级最高的任务。
最短作业优先 (SJF) / 最短剩余时间优先 (SRTF): 需要按预计运行时间排序，优先队列能快速取出时间最短的任务。
最早截止时间优先 (EDF - Earliest Deadline First): 实时系统中常用，需要按截止时间排序，优先队列能迅速找到最紧急的任务。
多级反馈队列 (Multilevel Feedback Queue): 虽然包含多个队列，但每个队列内部或队列间的调度往往依赖优先机制。
优先队列是“动态优先级”调度算法的标配，因为它能高效地处理不断变化的任务状态并快速选出最优解。但在顺序固定、平等轮转或预先规划的调度场景中，使用优先队列不仅多余，还会增加不必要的计算开销（ 
O(log⁡n) vs O(1)。
