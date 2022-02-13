# 控制结构

大多数大括号语言中的控制结构都可以在 `Solidity` 中使用。例如： if, else, while, do, for, break, continue, return 和一些 C 和 Javascript 中常见的语法。

Solidity 支持 try-catch 错误处理，但是只在 external 函数调用和合约创建调用。

用于表示条件的括号不可以被省略，但语句两边的花括号可以被省略。

注意，与 C 和 JavaScript 不同， Solidity 中非布尔类型数值不能被转换为布尔类型，因此 `if(1) {...}`的写法在 Solidity 中无效。