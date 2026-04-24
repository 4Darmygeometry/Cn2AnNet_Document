# Cn2AnNet 完整 API 文档（v1.0.0）

> ⚠️ 作者 (manyeyes) GitHub 账号仍活跃，但 Cn2AnNet 仓库已设为私有/删除，无官方文档。  
> 本文档通过 ILSpy 反编译 + 实际运行验证整理，为目前互联网上最完整的参考。  
> 本nuget包AOT友好，不会在AOT情形被裁剪导致缺功能

---

## 安装

```bash
dotnet add package Cn2AnNet
```

| 属性 | 值 |
|------|-----|
| 版本 | 1.0.0 |
| 作者 | manyeyes |
| 发布时间 | 2025-09-16 |
| 下载量 | ~361 |
| 目标框架 | .NET Core 3.1 / .NET Standard 2.0+ / .NET Framework 4.6.1+ / .NET 6.0 / .NET 8.0 / .NET 8.0 Android/iOS/MacCatalyst/Windows |
| 依赖 | **大多数框架无依赖**；.NET Framework 4.6.1 需 `System.ValueTuple (>= 4.3.0)` |

---

## 命名空间

```csharp
using Cn2AnNet;
```

---

## 一、Cn2An 类 — 中文数字 → 阿拉伯数字

### 构造函数

```csharp
public Cn2An()
```

### 核心方法

```csharp
public double Cn2AnConvert(object inputs, string mode = "strict")
```

| 参数 | 类型 | 说明 |
|------|------|------|
| inputs | object | 输入的中文数字字符串 |
| mode | string | 转换模式：`"strict"`（默认）、`"normal"`、 `"smart"` |

### Mode 说明

| 模式 | 说明 | 示例 | 结果 |
|------|------|------|------|
| `"strict"` | 严格模式，只接受标准中文数字（支持大写） | `"一百二十三"` | `123` |
| `"normal"` | ⚠️ **有 Bug**，纯数字串（如"一二三"）会崩溃 | `"一二三"` | ❌ 异常 |
| `"smart"` | 智能模式，支持阿拉伯+中文混合 | `"1百23"` | `123` |

### 实际验证示例

```csharp
var c2a = new Cn2An();

// ✅ strict 模式（默认）
c2a.Cn2AnConvert("一百二十三");           // 123.0
c2a.Cn2AnConvert("一百二十三", "strict"); // 123.0
c2a.Cn2AnConvert("负一百二十三");         // -123.0
c2a.Cn2AnConvert("一点二三");             // 1.23
c2a.Cn2AnConvert("壹佰贰拾叁", "strict");  // 123.0（大写）

// ✅ smart 模式
c2a.Cn2AnConvert("1百23", "smart");      // 123.0

// ❌ normal 模式 — 纯数字串会崩溃
c2a.Cn2AnConvert("一二三", "normal");    // KeyNotFoundException!
// 原因：源码 Bug，normal 模式 fallback 逻辑错误

// ❌ 不支持 direct 模式
c2a.Cn2AnConvert("零零三", "direct");    // ArgumentException: mode 仅支持 strict, normal, smart!
```

---

## 二、An2Cn 类 — 阿拉伯数字 → 中文数字

### 构造函数

```csharp
public An2Cn()
```

### 核心方法

```csharp
public string An2CnConvert(object inputs, string mode = "low")
```

| 参数 | 类型 | 说明 |
|------|------|------|
| inputs | object | 输入的数字（string 或 numeric） |
| mode | string | 转换模式：`"low"`（默认）、`"up"`、 `"rmb"`、`"direct"` |

### Mode 说明

| 模式 | 说明 | 示例 | 结果 |
|------|------|------|------|
| `"low"` | 小写中文数字 | `"123"` | `"一百二十三"` |
| `"up"` | 大写中文数字 | `"123"` | `"壹佰贰拾叁"` |
| `"rmb"` | 人民币格式 | `"123"` | `"壹佰贰拾叁元整"` |
| `"direct"` | 逐位原样转换 | `"012"` | `"零一二"` |

### 实际验证示例

```csharp
var a2c = new An2Cn();

// ✅ low 模式（默认）
a2c.An2CnConvert("123");           // "一百二十三"
a2c.An2CnConvert("123", "low");    // "一百二十三"
a2c.An2CnConvert("-123", "low");   // "负一百二十三"
a2c.An2CnConvert("1.23", "low");   // "一点二三"

// ✅ up 模式
a2c.An2CnConvert("123", "up");     // "壹佰贰拾叁"

// ✅ rmb 模式
a2c.An2CnConvert("123", "rmb");    // "壹佰贰拾叁元整"

// ✅ direct 模式
a2c.An2CnConvert("012", "direct");      // "零一二"
a2c.An2CnConvert("12.30", "direct");    // "一二点三零"
```

---

## 三、Transform 类 — 文本中的数字转换

### 构造函数

```csharp
public Transform()
```

### 核心方法

```csharp
public string TransformText(string inputs, string method = "cn2an")
```

| 参数 | 类型 | 说明 |
|------|------|------|
| inputs | string | 输入文本 |
| method | string | 转换方向：`"cn2an"`（默认）或 `"an2cn"` |

### 功能支持

| 功能 | cn2an 示例 | 结果 | an2cn 示例 | 结果 |
|------|-----------|------|-----------|------|
| 基础转换 | `"小王捡了一百块钱"` | `"小王捡了100块钱"` | `"小王捡了100块钱"` | `"小王捡了一百块钱"` |
| 日期 | `"二零零一年三月四日"` | `"二零零一年3月4日"` ⚠️ | `"2001年3月4日"` | `"二零零一年三月四日"` ✅ |
| 分数 | `"二分之一"` | `"1/2"` ✅ | `"1/2"` | `"二分之一"` ✅ |
| 百分比 | `"百分之五十"` | `"50%"` ✅ | `"50%"` | `"百分之五十"` ✅ |
| 摄氏度 | `"二十五摄氏度"` | `"25℃"` ✅ | `"25℃"` | `"二十五摄氏度"` ✅ |

### ⚠️ 已知问题

| 问题 | 说明 |
|------|------|
| 日期年份 Bug | `"二零零一年"` 年份部分未被转换，输出混合结果 `"二零零一年3月4日"` |
| 无 direct 参数 | Python 原版支持 `direct=True` 参数，C# 版不支持 |

### 实际验证示例

```csharp
var tf = new Transform();

// ✅ 基础转换
tf.TransformText("小王捡了一百块钱");           // "小王捡了100块钱"
tf.TransformText("小王捡了一百块钱", "cn2an");  // "小王捡了100块钱"
tf.TransformText("小王捡了100块钱", "an2cn");  // "小王捡了一百块钱"

// ⚠️ 日期 — 年份有 Bug
tf.TransformText("小王的生日是二零零一年三月四日", "cn2an");
// 实际输出: "小王的生日是二零零一年3月4日"
// 预期:     "小王的生日是2001年3月4日"
// Bug: 年份 "二零零一" 没被转换，月份和日期转换成功

tf.TransformText("小王的生日是2001年3月4日", "an2cn");
// 实际输出: "小王的生日是二零零一年三月四日" ✅

// ✅ 分数
tf.TransformText("抛出去的硬币为正面的概率是二分之一", "cn2an");
// "抛出去的硬币为正面的概率是1/2"

tf.TransformText("抛出去的硬币为正面的概率是1/2", "an2cn");
// "抛出去的硬币为正面的概率是二分之一"

// ✅ 百分比 — 无 Warning
tf.TransformText("百分之五十", "cn2an");       // "50%"

// ✅ 摄氏度 — 无 Warning
tf.TransformText("二十五摄氏度", "cn2an");     // "25℃"
```

### Warning 说明

Transform 的 `SubUtil` 方法用 try-catch 包裹转换逻辑：

```csharp
catch (Exception ex) {
    Console.WriteLine("Warning: " + ex.Message);
    return inputs;
}
```

**实际运行中，百分比和摄氏度转换无 Warning**，说明它们的内部转换逻辑未触发 `Cn2An` 的 normal 模式 Bug。

Warning 只在特定场景触发（如日期中的年份 `"二零零一"`），因为年份部分进入 `smart` 模式后 fallback 到 `normal` 模式，触发了 `ptnAllNum` 的 Bug。

---

## 四、Bug 推测分析（基于 ILSpy 逆向）

> ⚠️ 以下分析基于 ILSpy 反编译的 C# 源码推测，未经作者确认，可能存在偏差。

### 现象

`Cn2AnConvert("一二三", "normal")` 抛出 `KeyNotFoundException: '3' was not present in the dictionary`。

### 推测原因

从反编译代码 `CheckInputDataIsValid` 方法的 `ptnAllNum` fallback 逻辑观察：

```csharp
// 推测：此处返回了 DirectConvert 后的阿拉伯数字字符串
return (sign: item, integerData: DirectConvert(text3).ToString(), ..., isAllNum: true);
// 若 DirectConvert("一二三") = 123，则 integerData = "123"
```

上层 `Cn2AnConvert` 主流程随后将 `"123"` 再次传入 `DirectConvert`：

```csharp
if (flag) {  // isAllNum = true
    if (text2 == null) {
        num2 = DirectConvert(text.ToString());  // text = "123"
    }
}
// DirectConvert("123") 尝试查 _NUMBER_CN2AN["3"] → KeyNotFoundException
```

**推测结论**：`normal` 模式的 `ptnAllNum` fallback 可能错误地返回了已转换的阿拉伯数字字符串，而后续流程又将其当作中文数字处理，导致字典查询失败。

> 注：以上为基于反编译代码的推测，实际根因可能更复杂，需作者确认源码才能定论。

---

## 五、其他类

### Conf — 配置常量

存储数字映射字典，供内部使用。包含繁体转简体的大字典。

### Performance — 性能测试

```csharp
Performance.RunCn2AnTenThousandTimes();   // 测试 Cn2An 性能
Performance.RunAn2CnTenThousandTimes();   // 测试 An2Cn 性能
```

### Proces — 预处理

```csharp
Proces.Preprocess(data, pipelines);  // 文本预处理（繁体转简体、全角转半角等）
```

---

## 六、完整使用示例

```csharp
using Cn2AnNet;
using static System.Console;

class Program
{
    static void Main()
    {
        var c2a = new Cn2An();
        var a2c = new An2Cn();
        var tf = new Transform();

        // 中文 → 阿拉伯
        WriteLine(c2a.Cn2AnConvert("一百二十三"));           // 123
        WriteLine(c2a.Cn2AnConvert("负一千零二十四"));        // -1024
        WriteLine(c2a.Cn2AnConvert("三点一四"));              // 3.14
        WriteLine(c2a.Cn2AnConvert("壹佰贰拾叁", "strict"));  // 123
        WriteLine(c2a.Cn2AnConvert("1百23", "smart"));      // 123

        // 阿拉伯 → 中文
        WriteLine(a2c.An2CnConvert("123"));                 // 一百二十三
        WriteLine(a2c.An2CnConvert("123", "up"));           // 壹佰贰拾叁
        WriteLine(a2c.An2CnConvert("123", "rmb"));          // 壹佰贰拾叁元整
        WriteLine(a2c.An2CnConvert("-123", "low"));         // 负一百二十三
        WriteLine(a2c.An2CnConvert("1.23", "low"));         // 一点二三
        WriteLine(a2c.An2CnConvert("012", "direct"));      // 零一二

        // 文本转换
        WriteLine(tf.TransformText("小王捡了一百块钱"));              // 小王捡了100块钱
        WriteLine(tf.TransformText("小王捡了100块钱", "an2cn"));      // 小王捡了一百块钱
        WriteLine(tf.TransformText("概率是二分之一", "cn2an"));     // 概率是1/2
        WriteLine(tf.TransformText("概率是1/2", "an2cn"));           // 概率是二分之一
        WriteLine(tf.TransformText("百分之五十", "cn2an"));          // 50%
        WriteLine(tf.TransformText("二十五摄氏度", "cn2an"));        // 25℃
    }
}
```

---

## 七、与 Python 原版 cn2an 的差异

| 功能 | Python 原版 | Cn2AnNet | 差异 |
|------|------------|---------|------|
| `cn2an.cn2an("一二三", "normal")` | `123` | ❌ 崩溃 | **Bug** |
| `cn2an.cn2an("零零三", "direct")` | `"003"` | ❌ 不支持 | 缺少 direct 模式 |
| `cn2an.transform(..., direct=True)` | 支持 | ❌ 不支持 | 缺少 direct 参数 |
| `cn2an.an2cn("123", "low")` | `"一百二十三"` | ✅ 完美 | 一致 |
| 日期转换 | `"2001年3月4日"` | `"二零零一年3月4日"` | 年份 Bug |

---

## 八、作者信息

| 项目 | 信息 |
|------|------|
| NuGet 作者 | manyeyes |
| GitHub 主页 | [github.com/manyeyes](https://github.com/manyeyes) ✅ 仍活跃 |
| Cn2AnNet 仓库 | ❌ 已私有/删除 |
| 其他项目 | ManySpeech（语音识别）、音频处理等 |

---

## ⚠️ 重要提示

1. **仓库状态**：作者 GitHub 账号仍活跃，但 Cn2AnNet 仓库已设为私有/删除
2. **已知 Bug**：
   - `Cn2An.normal` 模式处理纯数字串会崩溃
   - `Transform.cn2an` 日期年份转换有 Bug（输出混合结果）
3. **无维护保障**：NuGet 包 v1.0.0 自 2025-09-16 后无更新
4. **安全性未知**：无法审计源码，生产环境请谨慎评估
5. **法律风险**：本包为闭源 DLL，反编译后修改重发布可能涉及版权风险。如需修复 Bug，建议：
   - 联系作者 manyeyes 获取源码授权
   - 或基于 Python `cn2an` (Ailln/cn2an) 自行独立实现

---

*本文档由 ILSpy 反编译 + 实际运行验证整理。*
*更新日期：2026-04-24*
