---
title: 为 WPF 的 ScrollViewer 添加滚动动画
date: 2024-08-01
categories:
- .NET
tags:
- WPF
---

在 WPF 中，ScrollViewer 默认没有直接支持滚动动画的属性。不过，我们可以通过自定义附加属性和动画来实现滚动动画效果。 

定义附加属性：
```C#

public static class ScrollViewerBehavior
{
    public static readonly DependencyProperty HorizontalOffsetProperty =
        DependencyProperty.RegisterAttached("HorizontalOffset", typeof(double), typeof(ScrollViewerBehavior),
            new UIPropertyMetadata(0.0, OnHorizontalOffsetChanged));

    public static void SetHorizontalOffset(FrameworkElement target, double value) =>
        target.SetValue(HorizontalOffsetProperty, value);

    public static double GetHorizontalOffset(FrameworkElement target) =>
        (double)target.GetValue(HorizontalOffsetProperty);

    private static void OnHorizontalOffsetChanged(DependencyObject target, DependencyPropertyChangedEventArgs e) =>
        (target as ScrollViewer)?.ScrollToHorizontalOffset((double)e.NewValue);

    public static readonly DependencyProperty VerticalOffsetProperty =
DependencyProperty.RegisterAttached("VerticalOffset", typeof(double), typeof(ScrollViewerBehavior),
new UIPropertyMetadata(0.0, OnVerticalOffsetChanged));

    public static void VerticalOffset(FrameworkElement target, double value) =>
        target.SetValue(VerticalOffsetProperty, value);

    public static double GetVerticalOffset(FrameworkElement target) =>
        (double)target.GetValue(VerticalOffsetProperty);

    private static void OnVerticalOffsetChanged(DependencyObject target, DependencyPropertyChangedEventArgs e) =>
        (target as ScrollViewer)?.ScrollToVerticalOffset((double)e.NewValue);

}
```
设置动画：
```xml
<Storyboard x:Key="ScrollStoryboard">
    <DoubleAnimation Storyboard.TargetName="ScrollViewer"
                     Storyboard.TargetProperty="(YourNamespace:ScrollViewerBehavior.HorizontalOffset)"
                     From="0" To="500" Duration="0:0:0.6">
        <DoubleAnimation.EasingFunction>
            <CircleEase EasingMode="EaseOut"/>
        </DoubleAnimation.EasingFunction>
    </DoubleAnimation>
</Storyboard>

```