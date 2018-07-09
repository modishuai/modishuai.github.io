---

title: WPF 实现图片的缩放、平移、旋转效果
categories:

- WPF
tags:
- WPF
- RenderTransfor

---

## RenderTransform 特效类

1. TranslateTransform：实现平移效果。
2. RotateTransform：实现旋转效果。
3. ScaleTransform：实现缩放效果。
4. SkewTransform：实现扭曲效果。
5. TransformGroup：特效组，让对象实现一组效果，包括对象的缩放、旋转、扭曲等变化效果合并起来使用。
6. MatrixTransform：通过矩阵算法实现更为复杂的变形，可以认为是以上特效的超集。

## 触控事件代码

```csharp
ManipulationStarting="Window_ManipulationStarting"
ManipulationDelta="Window_ManipulationDelta"
ManipulationInertiaStarting="Window_InertiaStarting"
```

```csharp
#region 触控事件
void Window_ManipulationStarting(object sender, ManipulationStartingEventArgs e)
{
  e.ManipulationContainer = this;
  e.Handled = true;
}

void Window_ManipulationDelta(object sender, ManipulationDeltaEventArgs e)
{
  Image imgToMove = e.OriginalSource as Image;
    imgsMatrix = ((MatrixTransform)imgToMove.RenderTransform).Matrix;
  imgsMatrix.RotateAt(e.DeltaManipulation.Rotation, e.ManipulationOrigin.X,         e.ManipulationOrigin.Y);
  imgsMatrix.ScaleAt(e.DeltaManipulation.Scale.X, e.DeltaManipulation.Scale.X, e.ManipulationOrigin.X,
  e.ManipulationOrigin.Y);
  imgsMatrix.Translate(e.DeltaManipulation.Translation.X, e.DeltaManipulation.Translation.Y);

imgToMove.RenderTransform = new MatrixTransform(imgsMatrix);
Rect containingRect = new Rect(((FrameworkElement)e.ManipulationContainer).RenderSize);
Rect shapeBounds = imgToMove.RenderTransform.TransformBounds(new Rect(imgToMove.RenderSize));
if (e.IsInertial && !containingRect.Contains(shapeBounds))
{
  e.Complete();
}
  e.Handled = true;
}

void Window_InertiaStarting(object sender, ManipulationInertiaStartingEventArgs e)
{
e.TranslationBehavior.DesiredDeceleration = 10.0 * 96.0 / (1000.0 * 1000.0);
e.ExpansionBehavior.DesiredDeceleration = 0.1 * 96 / (1000.0 * 1000.0);
e.RotationBehavior.DesiredDeceleration = 720 / (1000.0 * 1000.0);
e.Handled = true;
}
#endregion
```

## 鼠标事件

```csharp
 <Image Name="CurrentPhoto"  
        IsManipulationEnabled="True" 
        RenderOptions.BitmapScalingMode="HighQuality"
        MouseWheel="Image_MouseWheel" 
        PreviewMouseLeftButtonDown="Image_MouseLeftButtonDown" 
        PreviewMouseMove="Image_MouseMove" >
   <Image.RenderTransform>
     <MatrixTransform x:Name="transForm" />
   </Image.RenderTransform>
</Image>
```

```csharp
 #region 鼠标事件
 private void Image_MouseWheel(object sender, MouseWheelEventArgs e)
 {
   var center = getPosition(sender, e);
   var scale = (e.Delta > 0 ? 1.2 : 1 / 1.2);
   var matrix = transForm.Matrix;
   matrix.ScaleAt(scale, scale, center.X, center.Y);
   transForm.Matrix = matrix;
 }


private void Image_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
  dragStart = getPosition(sender, e);
}

private void Image_MouseMove(object sender, MouseEventArgs e)
{
  if ((e.LeftButton != MouseButtonState.Pressed))
  {
    return;
  }

  var current = getPosition(sender, e);
  var offset = current - dragStart;

  var matrix = transForm.Matrix;
  matrix.Translate(offset.X, offset.Y);

  transForm.Matrix = matrix;

  dragStart = current;
}

Point getPosition(object sender, MouseEventArgs e)
{
  return e.GetPosition(sender as UIElement) * transForm.Matrix;
}
#endregion
```
