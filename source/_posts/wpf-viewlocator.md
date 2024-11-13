---
title: 在WPF上使用ViewLocator来自动定位View
date: 2024-11-13
categories:
- .NET
tags:
- WPF
---

在 Avalonia 中，[ViewLocator](https://docs.avaloniaui.net/zh-Hans/docs/concepts/view-locator) 是一个用于解析与特定视图模型对应的视图（用户界面）的机制。  
例如，给定一个名为 `MyApplication.ViewModels.ExampleViewModel` 的视图模型，视图定位器将查找一个名为 `MyApplication.Views.ExampleView` 的视图。
视图定位器通常与DataContext属性一起使用，该属性用于将视图与其视图模型关联起来。

而在WPF中，我们也可以通过继承ResourceDictionary，创建 DataTemplate 来实现这个功能。
## 创建 ViewLocator 
```C#
public class ViewLocator: ResourceDictionary
{
    public static ViewLocator Instance = new ViewLocator();

    private ViewLocator()
    {
    }

    public DataTemplate Locate<TVM,TView>()
    {
        return Locate(typeof(TVM),typeof(TView));
    }

    public DataTemplate Locate(Type t1,Type t2)
    {
        DataTemplate dt = new DataTemplate()
        {
            DataType = t1,
        };
        dt.VisualTree = new FrameworkElementFactory(t2);
        try
        {
            this.Add(dt.DataTemplateKey, dt);

        }
        catch (System.ArgumentException)
        {
            //nothing
        }
        return dt;
    }
}
```
## 使用 ViewLocator 
关联 View 和 ViewModel
```C#
ViewLocator.Instance.Locate<FooViewModel,FooView>();
```
放置一个 ContentControl 到想要切换视图的区域，或者直接对 Window 使用。
```xml
<ContentControl x:Name="content_area" Resources="{x:Static local:ViewLocator.Instance}" />
```
切换视图
```
content_area.Content = new FooViewModel();
```
如果一切正常，此时 ContentControl 里显示的就是 FooView 控件的内容，而且 FooView 的 DataContext 也会自动关联 FooViewModel 。  