---
title: animation
---

#### 隐式动画
* layer的可动画属性的值被改变时，并非立刻在屏幕上体现出来，而是在下一次runloop中，以动画形式平滑过渡到新值。`CATransaction`管理了一系列的属性改变事件，在runloop周期中自动执行动画。UIView动画的begin&commit中会自动插入`CATransaction`的begin&commit。
* UIView动画禁用了隐式动画。UIView是关联的layer的代理，通过给代理方法 `actionForLayer:(CALayer *)layer forKey:(NSString *)event` 返回nil来实现。也可以通过重写 `actionForKey:(NSString *)event` 方法。
* CALayer属性值的改变是立即生效的，只是并未在界面上立即呈现。`presentationLayer` 保存了当前显示的layer的属性值，而 `modelLayer` 则保存了即将生效的属性值。`hitTest` 方法检测点击的位置。

#### 显式动画
* `CAAnimation`动画在执行完后，会自动回复到原始状态。这是由于在添加animation之前， `modelLayer` 控制 `presentationLayer` 去显示，而添加了animation之后，控制权交给了animation，`toValue` 只是提供给动画的一个目标值，并没有改变 `modelLayer` 的值，在动画结束后，animation交还控制权给 `modelLayer`，而 `modelLayer` 的状态仍保持在动画开始前的时刻，所以此时就会控制 `presentationLayer` 重新回到起点。通过代理方法 `animationDidStop` 中使用CATransaction禁用actions以及设置对应key的目标value来达成最终状态。也可以设置 **`animation.fillMode`**, **`animation.removeOnCompletion`** 来实现。
* `CAKeyFrameAnimation` 可以通过 `rotationMode` 来控制运动方向沿着切线
* `CASpringAnimation` 弹簧动画，各个属性如下：  
    `mass` : 首次到达目标点后的摆动幅度  
    `damping` : 阻尼值。阻尼越大，可摆动次数越少  
    `stiffness` : 类似于弹簧的松紧度。值越小(弹簧越松)，摆动速度越小；反之，值越大，摆动速度越大
* 虚拟属性。`CALayer.transform`有三个虚拟属性：`rotation` `position` `scale`。假如只需要旋转动画，可以使用`[CABasicAnimation animationWithKeyPath:@"transform.rotation"]`，`toValue`设置角度即可。如果直接修改`transform`整体，会影响其他的值。`position` 和 `scale`的值无法直接使用。
* `CAAnimationGroup`将不同的动画实例同时执行，如果动画实例定义了`duration`则该动画以此为准，未定义的使用group的`duration`
* `CATransition`是应用于图层整体的效果，也可以应用于单个属性，比如设置`layer.actions = @{@"propertyKey": transition}`。应用于整体时，需要将transition添加到对应`view.layer`上。例如：  
{% highlight javascript %}
- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController *)viewController  
{  
  ￼//set up crossfade transition  
  CATransition *transition = [CATransition animation];  
  transition.type = kCATransitionFade;  
  //apply transition to tab bar controller's view  
  [tabBarController.view.layer addAnimation:transition forKey:nil];  
}  
{% endhighlight %}  
此外，`UIView`的类方法也有对应的transition动画

#### 动画时间
* `CAMediaTiming`协议是`CALayer`、`CAAnimation`都遵循的协议。协议定义了如下属性：  
  `beginTime` ：动画执行前的延迟时间，默认为0，立即开始  
  `timeOffset` ：快进动画，从该时刻开始执行动画，忽略0到timeOffset之间的动画过程。结合`speed = 0`，可用来指定动画的状态  
  `speed` ：速度。假如`duration = 10, speed = 2`，那么动画将会加速执行，在**5s**时执行结束  
  `repeatCount` ：重复次数。使用`INFINITY`来设置无限重复
  `repeatDuration` ： 重复时间。可能与`repeatCount`相互冲突
  `autoReverses` ：动画结束后，是否会自动反向播放动画
  `fillMode` ：定义了动画结束后的行为。可用值包括：
	* `kCAFillModeForwards` : 保持最终状态
	* `kCAFillModeBackwards` : 结束后返回初始状态。另一个作用是，如果animation有`fromValue`，假如延迟**3s**开始动画，那么当动画加入layer后，会先变为`fromValue`的状态，然后**等待3s**后向`toValue`执行动画
	* `kCAFillModeBoth` : 结合前两者的行为
	* `kCAFillModeRemoved` : 默认值
* `speed`、`timeOffset`、`beginTime`作用在图层上时可用于动画的暂停和重启

#### 动画速度(缓冲函数)
* `CAMediaTimingFunction`缓冲函数类控制动画的速度变化。默认值是`kCAMediaTimingFunctionLinear`。`UIView`动画中的`option`默认值则是`UIViewAnimationOptionCurveEaseInOut`
* 自定以缓冲函数通过提交两个控制点，构成一个三次贝塞尔曲线函数
* 将连续动画时间以每秒60帧拆分，保持**初始值**和**终值**不变，根据**time**，对每一个单元计算中间值。如果**time**等差递增，那么结果就是匀速运动。使用函数处理**time**则会影响动画的行为  
{% highlight javascript %}
//弹性球自由落体的时间处理缓冲函数
float bounceEaseOut(float t)
{
  if (t < 4/11.0) {
    return (121 * t * t)/16.0;
  } else if (t < 8/11.0) {
      return (363/40.0 * t * t) - (99/10.0 * t) + 17/5.0;
  } else if (t < 9/10.0) {
      return (4356/361.0 * t * t) - (35442/1805.0 * t) + 16061/1805.0;
  }
  return (54/5.0 * t * t) - (513/25.0 * t) + 268/25.0;
}
{% endhighlight %}
