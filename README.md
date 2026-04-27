# Algorithm
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

这道题有点意思，其他几个操作都没啥，关键是这个跳转，你要跳转到合适的字节去，我当时想法就是遍历变cmds字符串，然后找到每一个op的位置字节号是多少保存起来，对应的字符串index也保存起来。其实在这里应该能找到规律了，
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


## 题目4 计算任务运行总耗时  
**(1)带依赖的任务调度问题**  
 带资源约束的依赖任务调度问题  虽然是调度，但是是拓扑排序

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

## 题目5 计算任务运行总耗时  
**多服务器在线任务调度问题**  
某机房有 serverNum 台规格相同依次编号的计算服务器，每台服务器同时处理的计算任务内存总量均不能超  
过 maxMemory 。现给定若干计算任务tasks，tasks[i]=[memory,cost］分别表示一个任务所需的内存和耗时。请把这些  
任务部署到服务器上进行处理，最后输出完成时长。部署规则：依次部署每一个计算任务，当空闲内存不满足要求  
时，阻塞等待。当有多台服务器可满足该任务的内存要求时，优先部署到编号小的服务器上。  
一个任务不能跨服务器部署，但一台服务器上能同时部署多个任务。  
```java
package org.example;




import java.util.*;

public class TaskSchedulerV2 {

    // 任务类：存储任务的内存需求和执行耗时
    static class Task {
        final int memory; // 任务所需内存
        final int cost;   // 任务执行耗时
        public Task(int memory, int cost) {
            this.memory = memory;
            this.cost = cost;
        }
    }

    // 运行中的任务类：记录任务的结束时间和内存占用
    static class RunningTask {
        final int endTime; // 任务结束的时间点
        final int memory;  // 任务占用的内存
        public RunningTask(int endTime, int memory) {
            this.endTime = endTime;
            this.memory = memory;
        }
    }

    // 服务器类：模拟服务器的内存管理和任务调度
    static class Server {
        final int maxMemory; // 服务器最大内存容量
        int usedMemory = 0;  // 当前已使用的内存
        // 优先队列（最小堆），按任务结束时间排序，用于快速获取最早结束的任务
        final PriorityQueue<RunningTask> runningTasks = new PriorityQueue<>(Comparator.comparingInt(t -> t.endTime));

        public Server(int maxMemory) {
            this.maxMemory = maxMemory;
        }

        // 释放当前时间点已完成的任务，回收内存
        public void releaseFinished(int currentTime) {
            // 循环检查堆顶（最早结束）的任务是否已完成
            while (!runningTasks.isEmpty() && runningTasks.peek().endTime <= currentTime) {
                usedMemory -= runningTasks.poll().memory; // 移除已完成任务并回收内存
            }
        }

        // 判断服务器是否能容纳指定内存的任务
        public boolean canFit(int memory) {
            return usedMemory + memory <= maxMemory;
        }

        // 将任务分配给服务器执行
        public void assign(int startTime, int cost, int memory) {
            runningTasks.offer(new RunningTask(startTime + cost, memory)); // 将任务加入优先队列
            usedMemory += memory; // 增加已用内存
        }

        // 获取服务器中最早结束的任务的结束时间
        public int getNextEndTime() {
            return runningTasks.isEmpty() ? Integer.MAX_VALUE : runningTasks.peek().endTime;
        }
    }

    // 调度任务的主方法
    public static int scheduleTasks(int serverNum, int maxMemory, List<Task> tasks) {
        // 创建服务器列表
        List<Server> servers = new ArrayList<>();
        for (int i = 0; i < serverNum; i++) {
            servers.add(new Server(maxMemory));
        }

        int currentTime = 0;       // 当前时间
        int globalFinishTime = 0;  // 所有任务完成的总时间

        // 遍历每个任务，尝试分配
        for (Task task : tasks) {
            // 如果任务所需内存超过单台服务器最大内存，无法执行
            if (task.memory > maxMemory) {
                throw new IllegalArgumentException("Task memory exceeds server capacity");
            }

            boolean assigned = false;
            // 循环直到任务被成功分配
            while (!assigned) {
                // 释放所有服务器上已完成的任务
                for (Server s : servers) s.releaseFinished(currentTime);

                // 寻找能容纳当前任务的服务器
                Server chosen = null;
                for (Server s : servers) {
                    if (s.canFit(task.memory)) {
                        chosen = s;
                        break;
                    }
                }

                // 如果找到合适的服务器，分配任务
                if (chosen != null) {
                    int finish = currentTime + task.cost; // 计算任务完成时间
                    chosen.assign(currentTime, task.cost, task.memory); // 分配任务
                    globalFinishTime = Math.max(globalFinishTime, finish); // 更新总完成时间
                    assigned = true;
                } else {
                    // 如果没有服务器能容纳，跳转到下一个任务结束的时间点
                    int nextTime = servers.stream()
                            .mapToInt(Server::getNextEndTime)
                            .min()
                            .orElse(Integer.MAX_VALUE);
                    currentTime = nextTime;
                }
            }
        }

        return globalFinishTime;
    }

    public static void main(String[] args) {
        // 测试用例
        List<Task> tasks = Arrays.asList(
                new Task(6, 5),
                new Task(5, 3),
                new Task(4, 2),
                new Task(8, 4)
        );

        int result = scheduleTasks(2, 10, tasks);
        System.out.println("总完成时长: " + result); // 7
    }
}

```

怎么想到定义个运行类RunningTask   放在优先队列里面  
在任务调度或模拟系统中，我们通常需要区分“静态的任务需求”（比如：我要跑多久、占多少内存）和“动态的运行状态”（比如：我几点几分结束、我现在占着哪台机器的内存）。  

**分离“静态需求”与“动态状态”**  
Task 类：是静态的。它只代表“需求”。比如：“我需要 5GB 内存，跑 10 秒”。它不知道自己在什么时候运行，也不知道自己在哪台服务器上。  
RunningTask 类：是动态的。它是 Task 在特定时间点、特定服务器上的实例化状态。它包含了“结束时间”（startTime + cost）。  
原因：如果不定义这个类，你就需要在其他地方（比如 Server 类里）维护额外的变量来记录“这个任务什么时候结束”，这会让代码变得混乱。将 endTime 封装在 RunningTask 里，符合面向对象编程的“高内聚”原则。  

**当前时间 = 所有服务器中，最早结束任务的结束时间  这种推进，一般都是用while 来搞么**  
因为你的任务调度逻辑本质上是一个“阻塞-重试”模型。  
想象你在餐厅等位：  
你去前台问：“有桌吗？”（尝试分配）  
前台说：“满了。”（分配失败）  
你不能走（不能处理下一个任务），你也不能傻站着（不能死循环空转）。  
你必须“等到”有人吃完走出来（时间跳跃）。  
人一走出来，你“再次”去问前台：“现在有桌了吗？”（重试）  
这个“检查 -> 等待 -> 再检查”的过程，天然就是一个循环。    

**currentTime = 0; // 需要一个全局时间变量 这个我怎么能想到呢**  
记住这个口诀：  
有耗时，必有轴；  
要模拟，先计时。  
你之所以觉得“没想到”，是因为我们平时写代码大多是“处理数据”（比如遍历一个列表算总和），数据是静止的。但在这种调度题目里，你是在“模拟过程”，数据是流动的。  
触发点：  
只要题目里出现了“耗时”、“开始时间”、“结束时间”、“等待”这些词，你就要立刻反应过来：“我是导演，我手里必须拿着一个记录当前时间的场记板。”  
触发点：  
只要你需要“计算未来”（比如 startTime + cost），你就必须先知道“现在”。这个“现在”必须存在一个变量里。  
怎么训练这种直觉？  
下次看到题目，画个图。  
只要你的图画出来了“时间轴”，你就一定会想到定义这个变量。  
比如这道题，你在纸上画：  
时间轴： 0秒 -------- 3秒 -------- 5秒 -------- 8秒  
任务1： [====运行====]  
任务2：             [====运行====]  
当你画出这条横线（时间轴）的时候，你的笔尖指在哪里，哪里就是 currentTime。  
这时候你的手就会自然地写下：  
// 我需要一支笔来指着当前时间  
int currentTime = 0;  

**总结**
你不是想不到，你是把这道题当成了“数学计算题”（算总数），而忽略了它其实是“物理模拟题”（模拟过程）。
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


题目4 看到“依赖”两个字，脑子里马上弹出“拓扑排序”。  
题目5 看到“专用NPU”，脑子里马上弹出“独立队列”  

针对题目4有想法：
```java
for (int j = 0; j < n; j++) {
    if (finishTime[j] == -1 && tasks.get(j).depend == taskId) {
        readyQueue.offer(j);
    }
}
```
在这一步之后，比如readyQueue本来就有很多值，单独对依赖taskid的所有task加入，当NPU空闲时，选择处于ready状态的任务立即执行。现在就是先排任务去取npu，而不是先搜索按个npu空闲？  

你这一问直接切中了这段代码最隐蔽的逻辑漏洞——“谁在控制节奏？”。
简单直接的回答是：你的代码目前是“任务驱动”（任务排队等NPU），而不是“事件驱动”（NPU空闲了去抓任务）。
**如果题目变成“抢占式调度”（比如来了个高优先级任务，要把NPU上正在跑的任务踢下去）   这个代码怎么修改**  
```java
import java.util.*;

/**
 * 严谨版抢占式调度器
 * 特性：
 * 1. 支持任务在不同时间点到达 (arrivalTime)
 * 2. 支持优先级抢占 (Preemption)
 * 3. 支持依赖关系 (Dependency)
 * 4. 使用等待队列防止任务“瞬移”
 */
public class AdvancedPreemptiveScheduler {

    // --- 1. 任务定义 ---
    static class Task {
        int id;
        int arrivalTime; // 【新增】任务到达时间
        int type;        // NPU类型 1-6
        int priority;    // 优先级 (越大越高)
        int remain;      // 剩余耗时
        int depend;      // 依赖的任务ID (-1无依赖)

        public Task(int id, int arrivalTime, int type, int priority, int remain, int depend) {
            this.id = id;
            this.arrivalTime = arrivalTime;
            this.type = type;
            this.priority = priority;
            this.remain = remain;
            this.depend = depend;
        }

        @Override
        public String toString() {
            return "Task{id=" + id + ", arrive=" + arrivalTime + ", pri=" + priority + ", rem=" + remain + "}";
        }
    }

    // --- 2. 事件定义 ---
    static class Event implements Comparable<Event> {
        int time;
        int type;       // 0: 任务到达/就绪, 1: 任务执行完成
        Task task;

        public Event(int time, int type, Task task) {
            this.time = time;
            this.type = type;
            this.task = task;
        }

        @Override
        public int compareTo(Event o) {
            if (this.time != o.time) return Integer.compare(this.time, o.time);
            // 完成事件优先，确保资源先释放
            return Integer.compare(this.type, o.type);
        }
    }

    // --- 3. 调度主逻辑 ---
    public int schedule(List<Task> tasks) {
        // 等待队列：每个NPU维护一个排队区
        PriorityQueue<Task>[] waitingQueues = new PriorityQueue[7];
        for (int i = 1; i <= 6; i++) {
            // 排队时，优先级高的在前；如果优先级相同，早到的在前
            waitingQueues[i] = new PriorityQueue<>((a, b) -> {
                if (b.priority != a.priority) return b.priority - a.priority;
                return a.arrivalTime - b.arrivalTime; 
            });
        }

        Task[] npuCurrentTask = new Task[7]; // 当前正在跑的任务
        int[] npuFinishTime = new int[7];    // NPU预计空闲时间

        // 依赖管理
        int[] inDegree = new int[tasks.size()];
        List<List<Integer>> children = new ArrayList<>();
        for (int i = 0; i < tasks.size(); i++) children.add(new ArrayList<>());

        for (Task t : tasks) {
            if (t.depend != -1) {
                inDegree[t.id]++;
                children.get(t.depend).add(t.id);
            }
        }

        // 全局事件堆
        PriorityQueue<Event> eventQueue = new PriorityQueue<>();

        // 【关键修改】初始化：根据任务自带的 arrivalTime 加入事件堆
        for (Task t : tasks) {
            if (inDegree[t.id] == 0) {
                eventQueue.offer(new Event(t.arrivalTime, 0, t));
            }
        }

        int maxTime = 0;

        while (!eventQueue.isEmpty()) {
            Event e = eventQueue.poll();
            int currentTime = e.time;
            Task task = e.task;
            int type = task.type;

            if (e.type == 0) {
                // --- 事件类型 A: 任务到达 ---
                Task currentRunning = npuCurrentTask[type];

                if (currentRunning == null) {
                    // 1. NPU空闲，直接跑
                    startTask(task, currentTime, type, npuCurrentTask, npuFinishTime, eventQueue);
                } else if (task.priority > currentRunning.priority) {
                    // 2. 抢占！
                    // 计算旧任务跑了多久
                    int executedTime = currentTime - (npuFinishTime[type] - currentRunning.remain);
                    currentRunning.remain -= executedTime;
                    
                    // 旧任务去排队（注意：这里不生成事件，而是进等待队列）
                    waitingQueues[type].offer(currentRunning);

                    // 新任务上位
                    startTask(task, currentTime, type, npuCurrentTask, npuFinishTime, eventQueue);
                } else {
                    // 3. 抢不过，去排队
                    waitingQueues[type].offer(task);
                }

            } else {
                // --- 事件类型 B: 任务完成 ---
                maxTime = Math.max(maxTime, currentTime);
                npuCurrentTask[type] = null; // NPU释放

                // 唤醒下游任务
                for (int childId : children.get(task.id)) {
                    inDegree[childId]--;
                    if (inDegree[childId] == 0) {
                        Task childTask = tasks.get(childId);
                        // 【关键逻辑】下游任务的“实际到达时间” = Max(自身定义时间, 依赖完成时间)
                        // 但在这里，我们只关心它“现在”就绪了，所以事件时间设为 currentTime
                        eventQueue.offer(new Event(currentTime, 0, childTask));
                    }
                }

                // 尝试从等待队列捞任务
                Task next = waitingQueues[type].poll();
                if (next != null) {
                    startTask(next, currentTime, type, npuCurrentTask, npuFinishTime, eventQueue);
                }
            }
        }

        return maxTime;
    }

    // 辅助方法：启动任务
    private void startTask(Task task, int currentTime, int type, Task[] npuCurrentTask, int[] npuFinishTime, PriorityQueue<Event> eventQueue) {
        npuCurrentTask[type] = task;
        npuFinishTime[type] = currentTime + task.remain;
        eventQueue.offer(new Event(npuFinishTime[type], 1, task));
        // System.out.println("T=" + currentTime + ": 开始运行 " + task);
    }

    // --- 4. 测试入口 ---
    public static void main(String[] args) {
        AdvancedPreemptiveScheduler scheduler = new AdvancedPreemptiveScheduler();
        List<Task> tasks = new ArrayList<>();

        // --- 测试用例：迟到的高优先级任务 ---
        // 任务0: T=0到达, 低优先级, 耗时10
        tasks.add(new Task(0, 0, 1, 1, 10, -1));
        
        // 任务1: T=2到达, 高优先级, 耗时5  <-- 注意这里！它晚来了2秒
        tasks.add(new Task(1, 2, 1, 5, 5, -1));

        System.out.println("运行高级抢占调度测试...");
        int result = scheduler.schedule(tasks);
        
        System.out.println("最终完成时间: " + result);
        
        // 预期流程：
        // T=0: 任务0到达，NPU空闲 -> 任务0开始跑 (预计T=10完)
        // T=2: 任务1到达，NPU忙 -> 检查优先级 (5 > 1) -> 抢占！
        //      任务0被踢 (已跑2秒，剩8秒)，进等待队列
        //      任务1开始跑 (预计T=7完)
        // T=7: 任务1完成 -> NPU空闲 -> 捞等待队列 -> 任务0回来跑
        //      任务0继续跑 (剩8秒) -> 预计T=15完
        // T=15: 任务0完成
        
        if (result == 15) {
            System.out.println("测试通过！时间线逻辑严密。");
        } else {
            System.out.println("测试失败，结果: " + result);
        }
    }
}
```

这里有一道设计题 很有意思，培养自己的架构思维  

## 题目 实现负载均衡器接口  
请实现以下接口:
LoadBalanceSys(int procNum, int maxConnectNum)-初始化。 procNum 表示系统中进程数，进程 id 从0开始依  
次编号，初始时都为下线状态； maxConnectNum 表示系统最多可保留的连接数  
procOnline(int[] pids）-上线若干个进程，pids[i] 表示进程id。  
。注：已上线的进程再次上线不会改变其已有的连接。  
procOffline(int[] pids）-下线若干个进程，这些进程上所有已建立的连接会被删除。注：已下线的进程再次下线不做任何处理。  
dispatch(int ip, int port, int proto)-收到以三元组 <目的ip，目的端口号port，协议号proto>表示的一  
条报文，按如下规则将其分派到某进程处理，并返回该进程id:  
若不存在上线的进程，返回 -1;  
若已存在该三元组对应的连接，则分派给该连接所在的进程处理，  
否则选择一个进程新建连接，处理该报文。优先选择已建连接数最少的进程，若有多个，则选择进程id最小的。  
先建后删：新建连接后，若超过系统的最大连接数，需将最久未被分派报文的连接删除。  
**这种题目如何建模，如何选择合适的数据结构怎么思考**  

1 **高频筛选的维度，最好独立出来，不要藏在对象深处**。  

思考陷阱：  
你可能会想：“进程有在线/下线状态，所以在 Process 类里加个 boolean online 吧。” —— 这是新手直觉。  
高阶思维（动静分离）：  
问自己：“这个状态变动的频率高吗？我需要频繁地按这个状态去筛选吗？”  
如果题目里频繁出现“查找所有在线的进程”，这意味着“在线”是一个索引维度。  
如果你把 status 放在 Process 对象里，每次想找在线进程，都得把所有进程遍历一遍O(N)。
顿悟时刻：如果我单独用一个数组 idStatus 或者一个 Set<Process> onlineSet 来专门存“在线进程”，查找速度就是 
O(1)  
2 不要只想着存数据，要先想好怎么查。**每一个查询需求，都对应一个数据结构**。  
题目需求：  
“收到报文，看三元组是否存在” -> 需要一个以三元组为 Key 的 Map。 -> ipToId  
“进程下线，要删掉该进程下的所有连接” -> 需要一个以进程 ID 为 Key 的 Map，Value 是连接列表。 -> idToIp  
“选连接数最少的进程” -> 需要一个按连接数排序的结构。 -> TreeSet  
3. **选容器（确定类型）**  
需要排序吗？ -> TreeSet / PriorityQueue  
需要极速查找吗？ -> HashMap  
需要维护顺序（LRU）吗？ -> LinkedHashMap / LinkedList  
只是简单的计数或状态标记？ -> 数组 / 基本类型变量  
一句话建议：  
**下次写题前，强迫自己先不写类，先写注释。列出所有的“查询需求”，每一个查询需求背后，都站着一个等着被你定义的变量（数据结构）。**  
## 题目 WIFI覆盖范围  
服务集标识符（Service Set Identifier，以下简称标识符）是无线网络的标识，用来区分不同的无线网络。  
一办公场地安装了某品牌的一批WIFI路由器 wifiRouters ， wifiRouters[] = [x, y，r］ 表示一个路由器所在位置(x, y)  
及其覆盖半径r。一个路由器的WIFI覆盖范围为一个圆形区域（含圆周），圆心在其覆盖范围的其他路由器被这个路由器所覆盖。  
路由器标识符设置的规则如下：  
。一个路由器被设置标识符后，会发送广播（携带标识符）信号给其覆盖范围内的其它路由器；后者接收到广播信号  
后，自动设置相同标识符，再进一步发送广播；以此类推，实现更多的路由器自动设置相同标识符。  
·有一根无限长的物理网线，可以将任意两个路由器连接起来，则相当于两个路由器均进入了对方的信号覆盖范围，使  
得两个路由器设置成相同的标识符。  
请你：  
1）挑选一个路由器，启动设置标识符;  
2）可根据需要使用物理网线连接两个合适的路由器。  
使得设置成相同标识符的路由器个数最多，最后返回该个数。  
1 <= wifiRouters.length <= 100, 0 <= x<= 10000, 0 <= y <= 10000, 1 <= r <= 10000   

复制 输入: [5,6,2],[7,6, 1],[6, 5, 1],[4,3,2],[6,3, 1],[7,3, 1,[1, 5, 4],[2, 1,3]  
复制输出：7  
解释：把输入的七个路由器依次编号为 A、B、C、D、E、F、G、H  
A到B 有个箭头连线，表示A可以覆盖B（注：B 的圆心刚好在A的覆盖范围边界上），但 B 不能覆盖A。  
·同理，A可以覆盖C；E可以覆盖F，F也可以覆盖E。  




