# algorithm
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
