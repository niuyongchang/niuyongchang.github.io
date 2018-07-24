## [个人博客](https://niuyongchang.github.io/ "个人博客")

#### 隐式动画
* layer的可动画属性的值被改变时，并非立刻在屏幕上体现出来，而是在下一次runloop中，以动画形式平滑过渡到新值。CATransaction管理了一系列的属性改变事件，在runloop周期中自动执行动画。UIView动画的begin&commit中会自动插入CATransaction的begin&commit。
* UIView动画禁用了隐式动画。UIView是关联的layer的代理，通过给代理方法 `actionForLayer:(CALayer *)layer forKey:(NSString *)event` 返回nil来实现。也可以通过重写 `actionForKey:(NSString *)event' 方法。
