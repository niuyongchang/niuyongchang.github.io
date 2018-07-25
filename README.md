## [个人博客](https://niuyongchang.github.io/ "个人博客")

#### 隐式动画
* layer的可动画属性的值被改变时，并非立刻在屏幕上体现出来，而是在下一次runloop中，以动画形式平滑过渡到新值。CATransaction管理了一系列的属性改变事件，在runloop周期中自动执行动画。UIView动画的begin&commit中会自动插入CATransaction的begin&commit。
* UIView动画禁用了隐式动画。UIView是关联的layer的代理，通过给代理方法 `actionForLayer:(CALayer *)layer forKey:(NSString *)event` 返回nil来实现。也可以通过重写 `actionForKey:(NSString *)event` 方法。
* CALayer属性值的改变是立即生效的，只是并未在界面上立即呈现。`presentationLayer` 保存了当前显示的layer的属性值，而 `modelLayer` 则保存了即将生效的属性值。`hitTest` 方法检测点击的位置。

#### 显式动画
* `CAAnimation`动画在执行完后，会自动回复到原始状态。这是由于在添加animation之前， `modelLayer` 控制 `presentationLayer` 去显示，而添加了animation之后，控制权交给了animation，`toValue` 只是提供给动画的一个目标值，并没有改变 `modelLayer` 的值，在动画结束后，animation交还控制权给 `modelLayer`，而 `modelLayer` 的状态仍保持在动画开始前的时刻，所以此时就会控制 `presentationLayer` 重新回到起点。通过代理方法 `animationDidStop` 中使用CATransaction禁用actions以及设置对应key的目标value来达成最终状态。也可以设置 **`animation.fillMode`**, **`animation.removeOnCompletion`** 来实现。
* `CAKeyFrameAnimation` 可以通过 `rotationMode` 来控制运动方向沿着切线
* `CASpringAnimation` 弹簧动画，各个属性如下：
  `mass` : 到达目标点后的形变幅度
