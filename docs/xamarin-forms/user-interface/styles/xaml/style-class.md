---
title: Xamarin 样式类
description: Xamarin 样式类可以将多个样式应用于控件, 而无需采用样式继承。
ms.prod: xamarin
ms.assetid: 4762401E-2B48-48F1-B6E4-61F7AF8AA46F
ms.technology: xamarin-forms
author: davidbritch
ms.author: dabritch
ms.date: 01/30/2019
ms.openlocfilehash: 4a353d64f0e7e29da6c64f93b8554c3661f4d389
ms.sourcegitcommit: c9651cad80c2865bc628349d30e82721c01ddb4a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2019
ms.locfileid: "70228125"
---
# <a name="xamarinforms-style-classes"></a>Xamarin 样式类

[![下载示例](~/media/shared/download.png) 下载示例](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/userinterface-styles-basicstyles)

_Xamarin 样式类可以将多个样式应用于控件, 而无需采用样式继承。_

## <a name="create-style-classes"></a>创建样式类

可以通过将的[`Class`](xref:Xamarin.Forms.Style.Class) [`Style`](xref:Xamarin.Forms.Style)属性设置为表示类名的来`string`创建样式类。 此功能的优势在于, 通过使用`x:Key`属性定义显式样式, 可以将多个样式类应用[`VisualElement`](xref:Xamarin.Forms.VisualElement)于。

> [!IMPORTANT]
> 多个样式可以共享同一类名称, 前提是它们面向不同的类型。 这将启用多个具有相同名称的样式类以面向不同的类型。

下面的示例演示了[`BoxView`](xref:Xamarin.Forms.BoxView)三个样式类和[`VisualElement`](xref:Xamarin.Forms.VisualElement)一个样式类:

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        <Style TargetType="BoxView"
               Class="Separator">
            <Setter Property="BackgroundColor"
                    Value="#CCCCCC" />
            <Setter Property="HeightRequest"
                    Value="1" />
        </Style>

        <Style TargetType="BoxView"
               Class="Rounded">
            <Setter Property="BackgroundColor"
                    Value="#1FAECE" />
            <Setter Property="HorizontalOptions"
                    Value="Start" />
            <Setter Property="CornerRadius"
                    Value="10" />
        </Style>    

        <Style TargetType="BoxView"
               Class="Circle">
            <Setter Property="BackgroundColor"
                    Value="#1FAECE" />
            <Setter Property="WidthRequest"
                    Value="100" />
            <Setter Property="HeightRequest"
                    Value="100" />
            <Setter Property="HorizontalOptions"
                    Value="Start" />
            <Setter Property="CornerRadius"
                    Value="50" />
        </Style>

        <Style TargetType="VisualElement"
               Class="Rotated"
               ApplyToDerivedTypes="true">
            <Setter Property="Rotation"
                    Value="45" />
        </Style>        
    </ContentPage.Resources>
</ContentPage>
```

、和样式类分别将属性设置[`BoxView`](xref:Xamarin.Forms.BoxView)为特定值。 `Circle` `Rounded` `Separator`

`VisualElement`样式类[`TargetType`](xref:Xamarin.Forms.Style.TargetType)的为[,这意味着它只能应用于实例。`VisualElement`](xref:Xamarin.Forms.VisualElement) `Rotated` 但是, 其[`ApplyToDerivedTypes`](xref:Xamarin.Forms.Style.ApplyToDerivedTypes)属性设置为`true`, 这可确保它可应用于[`BoxView`](xref:Xamarin.Forms.BoxView)派生自`VisualElement`的任何控件, 例如。 有关将样式应用于派生类型的详细信息, 请参阅[将样式应用于派生类型](implicit.md#apply-a-style-to-derived-types)。

等效 C# 代码如下：

```csharp
var separatorBoxViewStyle = new Style(typeof(BoxView))
{
    Class = "Separator",
    Setters =
    {
        new Setter
        {
            Property = VisualElement.BackgroundColorProperty,
            Value = Color.FromHex("#CCCCCC")
        },
        new Setter
        {
            Property = VisualElement.HeightRequestProperty,
            Value = 1
        }
    }
};

var roundedBoxViewStyle = new Style(typeof(BoxView))
{
    Class = "Rounded",
    Setters =
    {
        new Setter
        {
            Property = VisualElement.BackgroundColorProperty,
            Value = Color.FromHex("#1FAECE")
        },
        new Setter
        {
            Property = View.HorizontalOptionsProperty,
            Value = LayoutOptions.Start
        },
        new Setter
        {
            Property = BoxView.CornerRadiusProperty,
            Value = 10
        }
    }
};

var circleBoxViewStyle = new Style(typeof(BoxView))
{
    Class = "Circle",
    Setters =
    {
        new Setter
        {
            Property = VisualElement.BackgroundColorProperty,
            Value = Color.FromHex("#1FAECE")
        },
        new Setter
        {
            Property = VisualElement.WidthRequestProperty,
            Value = 100
        },
        new Setter
        {
            Property = VisualElement.HeightRequestProperty,
            Value = 100
        },
        new Setter
        {
            Property = View.HorizontalOptionsProperty,
            Value = LayoutOptions.Start
        },
        new Setter
        {
            Property = BoxView.CornerRadiusProperty,
            Value = 50
        }
    }
};

var rotatedVisualElementStyle = new Style(typeof(VisualElement))
{
    Class = "Rotated",
    ApplyToDerivedTypes = true,
    Setters =
    {
        new Setter
        {
            Property = VisualElement.RotationProperty,
            Value = 45
        }
    }
};

Resources = new ResourceDictionary
{
    separatorBoxViewStyle,
    roundedBoxViewStyle,
    circleBoxViewStyle,
    rotatedVisualElementStyle
};
```

## <a name="consume-style-classes"></a>使用样式类

可以通过将控件[`StyleClass`](xref:Xamarin.Forms.NavigableElement.StyleClass) `IList<string>`的属性设置为样式类名称的列表来使用样式类。 如果控件的类型与样式类的[`TargetType`](xref:Xamarin.Forms.Style.TargetType)匹配, 则将应用样式类。

下面的示例演示三[`BoxView`](xref:Xamarin.Forms.BoxView)个实例, 每个实例都设置为不同的样式类:

```xaml
<ContentPage ...>
    <ContentPage.Resources>
        ...
    </ContentPage.Resources>
    <StackLayout Margin="20">
        <BoxView StyleClass="Separator" />       
        <BoxView WidthRequest="100"
                 HeightRequest="100"
                 HorizontalOptions="Center"
                 StyleClass="Rounded, Rotated" />
        <BoxView HorizontalOptions="Center"
                 StyleClass="Circle" />
    </StackLayout>
</ContentPage>    
```

在此示例中, 第[`BoxView`](xref:Xamarin.Forms.BoxView)一个的样式为行分隔符, 而第三个`BoxView`为循环。 第二`BoxView`个应用了两个样式类, 它为其舍入角并将其旋转45度:

![样式类的 BoxViews 样式](style-class-images/boxviews.png)

> [!IMPORTANT]
> 可以将多个样式类应用于控件, 因为[`StyleClass`](xref:Xamarin.Forms.NavigableElement.StyleClass)该属性的类型`IList<string>`为。 出现这种情况时, 将按升序顺序应用样式类。 因此, 当多个样式类设置相同的属性时, 位于最高列表位置的 style 类中的属性将优先。

等效 C# 代码如下：

```csharp
...
Content = new StackLayout
{
    Children =
    {
        new BoxView { StyleClass = new [] { "Separator" } },
        new BoxView { WidthRequest = 100, HeightRequest = 100, HorizontalOptions = LayoutOptions.Center, StyleClass = new [] { "Rounded", "Rotated" } },
        new BoxView { HorizontalOptions = LayoutOptions.Center, StyleClass = new [] { "Circle" } }
    }
};
```

## <a name="related-links"></a>相关链接

- [基本样式 （示例）](https://docs.microsoft.com/samples/xamarin/xamarin-forms-samples/userinterface-styles-basicstyles)
