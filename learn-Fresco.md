## 缩放 ##
focusCrop:Fresco 可以设置针对某个点的“centerCrop” 效果  
首先先在XML中指定缩放类型
~~~XML 
fresco:actualImageScaleType = "focusCrop"
~~~
之后可以在Java代码中设置显示的中心点
~~~Java
PointF focusPoint;
mSimpleDraweeView
  .getHierarchy()
  .setActualImageFocusPoint(focusPoint);
~~~
