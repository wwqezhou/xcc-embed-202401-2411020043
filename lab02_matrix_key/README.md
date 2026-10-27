# 实验 2：矩阵键盘

## 一、实验目的

1. 掌握矩阵键盘的行扫描原理与 GPIO 输入/输出配置；
2. 使用寄存器读取 `IDR`、写 `ODR` 完成 3×4 矩阵键盘扫描；
3. 实现按键消抖，并将键值（1~12）以 BCD 码形式通过 LED 显示。

## 二、实验环境

- 开发板：STM32F407 开发板
- 主控芯片：STM32F407ZET6（Cortex-M4，168 MHz）
- 开发工具：STM32CubeIDE
- 调试器：ST-Link（SWD）
- 工程文件：`code/0602_MatrixKey.ioc`

## 三、硬件连接

| 器件 | 引脚 | 说明 |
| --- | --- | --- |
| 列线 COL1~COL4 | PE0 ~ PE3 | 输入，内部上拉 |
| 行线 ROW1~ROW3 | PE4 ~ PE6 | 推挽输出 |
| 键值显示 LED | PF0 ~ PF3（个位）、PF4 ~ PF7（十位） | 推挽输出，低电平点亮 |

## 四、原理与设计

1. **行扫描**：每次只将一行（PE4~PE6 中的一根）输出低电平，其余行输出高电平。
2. **列检测**：读取 `GPIOE->IDR & 0x0F`，若不为 `0x0F` 说明该行有按键按下；
   根据哪一列为 0 判断列号。
3. **消抖**：检测到按下后延时 16 ms 再次确认，避免抖动误判。
4. **键值计算**：3 行 × 4 列 = 12 个键，`KeyVal = 4 * 行号 + 列号`（1~12）；
   行号通过数组 `PreLine[3] = {2, 0, 1}` 与实际行序对应。
5. **显示**：`GPIOF->ODR = ~((KeyVal / 10 << 4) + KeyVal % 10)`，
   PF4~PF7 显示十位、PF0~PF3 显示个位，LED 低电平点亮。

```c
for (i = 0; i < 3; i++)
{
  if ((GPIOE->IDR & 0x0F) != 0x0F)              /* 有键按下 */
  {
    HAL_Delay(16);                               /* 消抖 */
    if ((GPIOE->IDR & 0x0F) != 0x0F)
    {
      switch (GPIOE->IDR & 0x0F)
      {
        case 0x0E: KeyVal = 4 * PreLine[i] + 1; break;
        case 0x0D: KeyVal = 4 * PreLine[i] + 2; break;
        case 0x0B: KeyVal = 4 * PreLine[i] + 3; break;
        case 0x07: KeyVal = 4 * PreLine[i] + 4; break;
        default  : break;
      }
    }
  }
  GPIOE->ODR = ~(1 << (i + 4));                  /* 切换到下一行 */
}
GPIOF->ODR = ~((KeyVal / 10 << 4) + KeyVal % 10); /* BCD 显示 */
```

## 五、实验现象

按下任意按键后，PF 口 LED 以 BCD 码显示对应键值（1~12），
例如按下第 2 行第 3 列显示 "07"，松开后保持最后一次键值。

> 现象截图请放入 `screenshots/` 目录。

## 六、问题与解决

| 问题 | 解决方法 |
| --- | --- |
| 按键识别不稳定 | 增加 16 ms 延时消抖并在消抖后复测 |
| 键值与行列不对应 | 用 `PreLine[]` 校正行扫描顺序与物理行号的映射 |
| BCD 显示错位 | 十位左移 4 位后与个位相加，再按位取反输出 |

## 七、实验总结

本实验通过直接操作 `IDR`/`ODR` 寄存器完成矩阵键盘行扫描，掌握了 3×4 键盘的
键值计算与消抖方法，并实现键值的 BCD 码 LED 显示。

---

## 文件说明

| 目录/文件 | 说明 |
| --- | --- |
| `code/` | STM32CubeIDE 完整工程（源文件在 `code/Core/Src/`） |
| `report/` | 实验报告 |
| `screenshots/` | 实验现象截图 |
