# Unity 本地化管理系统 (Unity Language Manager)  
## 📌 简介  
这是一个轻量、高效的 Unity 本地化管理系统，专注于通过 **JSON 文件** 存储多语言字典，提供简洁的 API 实现文本动态切换。系统基于 `MonoSingleton` 设计，支持跨场景持久化，内置语言切换事件通知，适用于中小型项目的本地化需求。依赖 Newtonsoft.Json 处理 JSON 解析，确保灵活的文本读取与管理。  


## ✨ 核心功能  
- **多语言加载**：从 `StreamingAssets/lang` 目录读取 `{语言标识}.json` 文件（如 `en_US.json`），自动解析为键值对字典。  
- **语言切换**：支持动态切换语言，自动重新加载对应字典并触发全局事件通知。  
- **异常容错**：文件不存在或解析失败时输出详细错误日志，避免崩溃。  
- **事件订阅**：提供 `OnLanguageChanged` 事件，方便 UI 或其他组件监听语言变化并更新。  
- **持久化配置**：切换语言时自动保存到 `GameConfig`（需配合项目中的配置系统使用，可自定义）。  


## ⚙️ 核心设计  
- **单例模式**：继承 `MonoSingleton`，确保全局唯一实例且跨场景保留。  
- **资源管理**：语言字典存储在内存中，切换语言时清空旧数据避免冗余。  
- **轻量扩展**：可通过修改 `Load` 方法支持其他存储路径（如 `Resources` 文件夹）或格式（如 XML）。  


## 🚀 快速开始  

### 前置条件  
1. **Unity 版本**：2020.3 或更高（兼容 .NET Standard 2.1+）。  
2. **Newtonsoft.Json 依赖**：需通过 Unity 包管理器安装（安装方式见下文）。  


### 步骤 1：安装 Newtonsoft.Json  (项目自带)
本项目依赖 `Newtonsoft.Json` 解析 JSON，按以下步骤安装：  

1. 打开 Unity 编辑器，进入顶部菜单：`Window > Package Manager`（包管理器）。  
2. 点击右上角 `+` 按钮，选择 `Add package by name or Git URL...`。  
3. 输入以下 Git URL 并安装：  
   ```  
   com.unity.nuget.newtonsoft-json  
   ```  


### 步骤 2：配置语言文件  
1. 在项目 `Assets/StreamingAssets/lang` 目录下（无则手动创建），添加各语言的 JSON 文件，命名格式为 `{语言标识}.json`（如 `en_US.json`、`zh_CN.json`）。  
   - 示例 `en_US.json`：  
     ```json
     {
       "1": "Hello",
       "2": "World",
       "3": "English (United States of America)"
     }
     ```  
   - 示例 `zh_CN.json`：  
     ```json
     {
       "1": "你好",
       "2": "世界",
       "3": "简体中文（中国）"
     }
     ```  


### 步骤 3：基础使用  

```csharp
using Core.System;
using UnityEngine;

public class Test : MonoBehaviour
{
    void Start()
    {

        // 订阅语言变化事件（如更新UI文本）
        LanguageManager.Instance.Subscribe(OnLanguageChanged);
    }

    private void OnLanguageChanged()
    {
        Debug.Log("语言已切换！");
        // 示例：更新UI文本组件
        // someTextComponent.text = LanguageManager.Instance.Get("1");
    }

    void OnDestroy()
    {
        // 取消订阅（避免内存泄漏）
        LanguageManager.Instance.Unsubscribe(OnLanguageChanged);
    }
}
```  


### 步骤 4：获取本地化文本  
通过 `Get` 方法获取当前语言对应的翻译内容：  

```csharp
// 获取键 "1" 的翻译（当前语言为 en_US 时返回 "Hello"）
string greeting = LanguageManager.Instance.Get("1"); 

// 键不存在时返回原键（如 Get("UNKNOWN_KEY") 返回 "UNKNOWN_KEY"）
string unknownText = LanguageManager.Instance.Get("UNKNOWN_KEY"); 
```  


### 步骤 5：动态切换语言  
调用 `SetLanguage` 方法切换语言，系统会自动加载新文件并触发事件：  

```csharp
// 切换至中文（假设已存在 zh_CN.json）
LanguageManager.Instance.SetLanguage("zh_CN"); 
```  


## 🔧 扩展与自定义  
尽管系统设计简洁，仍支持以下扩展方向：  

### 修改存储路径  
当前语言文件存储在 `StreamingAssets/lang`，如需调整（如改用 `Resources` 文件夹），可修改 `Load` 方法中的 `path` 生成逻辑：  
```csharp
// 原路径：StreamingAssets/lang/{language}.json
string path = Path.Combine(Application.dataPath, "Resources/lang", $"{_language}.json");
```  


### 支持更多文件格式  
如需解析 XML 或 CSV，可重写 `Load` 方法，使用对应解析库（如 `XmlSerializer`）读取文件并转换为字典：  
```csharp
// 示例：替换为 XML 解析
string xmlPath = Path.Combine(..., $"{_language}.xml");
var xmlDoc = new XmlDocument();
xmlDoc.Load(xmlPath);
var dict = new Dictionary<string, string>();
foreach (XmlNode node in xmlDoc.SelectNodes("//entry"))
{
    dict[node.Attributes["key"].Value] = node.InnerText;
}
_languageDictionary.Merge(dict);
```  


### 自定义配置保存  
当前切换语言时自动保存到 `GameConfig`，如需适配项目自身的配置系统，可修改 `SetLanguage` 方法中的配置逻辑：  
```csharp
// 原逻辑：保存到 GameConfig.Instance.settings
// 替换为自定义配置类
MyProjectConfig config = MyProjectConfig.Load();
config.language = _language;
MyProjectConfig.Save(config);
```  


## 📄 许可证  
本项目采用 https://opensource.org/licenses/MIT，可自由修改、分发，无需公开衍生代码。  


## 🤝 贡献与反馈  
欢迎提交 Issue 报告问题或提出需求，也欢迎通过 Pull Request 贡献代码！  
- 问题反馈：https://github.com/YourUsername/Unity-Language-Manager/issues  


**提示**：首次使用建议参考 `Assets/StreamingAssets/lang` 下的示例 JSON 文件，确保格式正确！
