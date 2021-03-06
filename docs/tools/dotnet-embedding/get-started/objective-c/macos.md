---
title: MacOS 入门
description: 本文档介绍如何开始使用 macOS 的 .NET 嵌入。 它讨论了要求，并提供了一个示例应用程序，用于演示如何绑定托管程序集并在 Xcode 项目中使用生成的输出。
ms.prod: xamarin
ms.assetid: AE51F523-74F4-4EC0-B531-30B71C4D36DF
author: davidortinau
ms.author: daortin
ms.date: 11/14/2017
ms.openlocfilehash: f0e2128bca5d2965395647353cd5a95a4030439f
ms.sourcegitcommit: db422e33438f1b5c55852e6942c3d1d75dc025c4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/24/2020
ms.locfileid: "76725276"
---
# <a name="getting-started-with-macos"></a>MacOS 入门

## <a name="what-you-will-need"></a>需要的内容

* 按照[目标入门-C](~/tools/dotnet-embedding/get-started/objective-c/index.md)指南中的说明进行操作。

## <a name="hello-world"></a>Hello World

首先，在中C#生成一个简单的 hello world 示例。

### <a name="create-c-sample"></a>创建C#示例

打开 Visual Studio for Mac，创建名为 " **hello-从-csharp**" 的新 Mac 类库项目，并将其保存到 **~/Projects/hello-from-csharp**。

将**MyClass.cs**文件中的代码替换为以下代码片段：

```csharp
using AppKit;
public class MyNSView : NSTextView
{
    public MyNSView ()
    {
        Value = "Hello from C#";
    }
}
```

프로젝트를 빌드합니다. 生成的程序集将另存为 **~/Projects/hello-from-csharp/hello-from-csharp/bin/Debug/hello-from-csharp.dll**。

### <a name="bind-the-managed-assembly"></a>绑定托管程序集

有了托管程序集后，可通过调用 .NET 嵌入进行绑定。

如[安装](~/tools/dotnet-embedding/get-started/install/install.md)指南中所述，可以在项目中使用自定义 MSBuild 目标来完成生成后步骤，也可以手动执行此操作：

```shell
cd ~/Projects/hello-from-csharp
objcgen ~/Projects/hello-from-csharp/hello-from-csharp/bin/Debug/hello-from-csharp.dll --target=framework --platform=macOS-modern --abi=x86_64 --outdir=output -c --debug
```

将在 **~/Projects/hello-from-csharp/output/hello-from-csharp.framework**中放置该框架。

### <a name="use-the-generated-output-in-an-xcode-project"></a>在 Xcode 项目中使用生成的输出

打开 Xcode 并创建新的 Cocoa 应用程序。 将其命名为 " **hello-从-csharp** " 并选择 "**目标 C** " 语言。

在查找器中打开 **~/Projects/hello-from-csharp/output**目录，选择 " **hello**-"，将其拖到 "Xcode" 项目，并将其放置在项目中的 " **hello-csharp** " 文件夹的紧上方。

![拖放框架](macos-images/hello-from-csharp-mac-drag-drop-framework.png)

请确保在弹出的对话框中选中 "**需要时复制项**"，然后单击 "**完成**"。

![根据需要复制项](macos-images/hello-from-csharp-mac-copy-items-if-needed.png)

选择 " **hello-csharp** " 项目，并导航到 " **hello-csharp** " 目标的 "**常规**" 选项卡。在 "**嵌入的二进制文件**" 部分中，添加 " **hello-csharp**"。

![嵌入的二进制文件](macos-images/hello-from-csharp-mac-embedded-binaries.png)

打开**ViewController**，并将内容替换为：

```objc
#import "ViewController.h"

#include "hello-from-csharp/hello-from-csharp.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    MyNSView *view = [[MyNSView alloc] init];
    view.frame = CGRectMake(0, 200, 200, 200);
    [self.view addSubview: view];
}

@end
```

最后，运行 Xcode 项目，如下所示：

![在模拟器C#中运行的示例 Hello](macos-images/hello-from-csharp-mac.png)
