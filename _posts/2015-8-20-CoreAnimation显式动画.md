## 显示动画


隐式动画是在iOS平台创建动态用户界面的一种直接方式，也是UIKit动画机制的基础，不过它并不能涵盖所有的动画类型。在本章中，我们将研究显式动画的框架结构，实现方式，已经显式动画的类型。

IOS系统一个很重要的特色就是它的动画流畅，优美。为了实现这一点，苹果在这个系统中做了很多工作。其中，为了开发者能够快速方便的使用动画，苹果做了一系列的封装，为我们提供了各种不同的“动画武器”，满足开发者的基本需求。当我们选定合适的“动画武器”后，仅仅需要设置几个关键值（比如时间，开始，结束状态）就能实现基本的动画，下图是苹果为开发者封装设计的动画模块。

![CAAniamtion](https://github.com/Ambtion/ambtion.github.io/blob/master/imageSource/CoreAnimaiton/CoreAnimation/CAAnimaiton_Frameworlk.png?raw=ture)

在这个模块中，我们需要明确理解的知识点分别是:

* 1: CAAnimation是做什么的，它实现的NSCoding,CAMediaTiming,CAAction协议分别实现了什么功能
* 2: CAAnimaitonGrop，CAPropertyAnimaiton，CATranstion各自场景
* 3: CABAsicAnimaiton,CAKeyframeAnmation的各自场景

### CAAnimation

CAAnimation是一个动画的抽象类,他将动画的共性抽取，然后通过对一些协议的支持，构建动画的基本通道。CAAnimation仅仅提供动画通道，要实现动画，需要使用它的子类。

* 1: 对CAMediaTiming协议的支持，提供动画时间。像duration，beginTime和repeatCount这些时间相关的属性都在这个类中，这些属性结合起来，能够准确灵活的控制着动画时间(__详情可以参考文献controlling Animation Timing__)
* 2: 对CAAction协议的支持，提供获取动画方式的通道(__详情可以参看隐式动画篇幅__)
* 3: 对NSCoding的支持，满足CAAnimation的键值容器特性（个人猜测可能和动画获取的方式有关。同时这一属性能够使我们在开发中对每一个动画快速识别，比如为CAAnimation赋一个ID）
* 4: 定义CAAnimationDelegate,在动画开始或者结束的时候能够回调
* 5: 提供removedOnCompletion接口，用于标识动画是否该在结束后自动释放（默认YES，为了防止内存泄露）

		
###  CAPropertyAnimation

CAPropertyAnimation是一个CAAnimation的抽象子类，他主要用于参加针对CALayer属性操作的动画。CAPropertyAnimation有一个属性keyPath，这个一个字符串类型的属性，设定keyPath的值，就可以通过键值绑定CALayer的对应属性，从而实现动画。

####  CABAsicAnimation

CABAsicAnimation提供了一种针对CALayer单一属性实现动画的功能。（PS:在参阅CALayer的属性的时候，细心的话可能会注意到有些属性的结尾有一个Animatable的描述，这个其实就是表示CALayer支持该属性的动画）

CABAsicAnimation继承于CAPropertyAnimation，并添加了如下属性：
	
	id fromValue 
	id toValue 
	id byValue

fromValue代表了动画开始之前属性的值，toValue代表了动画结束之后的值，byValue代表了动画执行过程中改变的值。也就是fromValue +  byValue = toValue;所以为了防止冲突，我们不需要一次制定三个属性值。

fromValue， toValue， byValue之所以指定是__id__类型是因为它可以支持多中类型的数据结构:

	1: integers and doubles
	2: CGRect, CGPoint, CGSize, and CGAffineTransform 
	3: ATransform3D data structures
	4: CGColor and CGImage references
	
####  CAKeyframeAnmatoin

CAKeyframeAnmatoin主要提供了一种对动画关键帧控制的能力。

CAKeyframeAnmatoin提供了两个属性来控制动画关键帧

	1: a Core Graphics path（Path属性）  

	2: 一个数值的Array(Values属性)

Path属性主要适用CALayer的__anchorPoint__或者__position__这些CGPoint类型的属性。当设置Path属性,轨迹会按照Path的每一个点去执行动画。当Path属性设置，Values属性自动忽略。
同时在使用Path控制关键帧动画的时候，我们可以通过设置rotationMode属性，从而是Layer旋转从而匹配Path轨迹的移动。

Values属性适用与CALayer的任何属性的动画关键帧控制。例如：

* 1：提供一个CGImage对象组成的Array,可以控制CALayer的content按照给定内容的执行动画。
* 2：提供一个CATransform3D对象组成的Array,可以控制CALayer的transform按照给定内容执行动画


###  CATranstoin

CATranstoin主要用于转场动画的创建，它影响是整个CALayer，而非CALayer的一个属性变化
IOS系统公开了四种类型的CATranstoin动画，分别是

	kCATransitionFade
	kCATransitionMoveIn
	kCATransitionPush
	kCATransitionReveal

同时通过SubType来控制转场动画的方向，分别是

	kCATransitionFromRight
	kCATransitionFromLeft
	kCATransitionFromTop
	kCATransitionFromBottom

另外CATranstoin提供了一个自动以转场动画的接口。filter(滤镜接口)。使用该功能需要注意的是：

* 1：Core Image 是在IOS5及其以后公开的，所以在IOS5以前使用滤镜接口无效，也可以理解为IOS系统上该功能支持5及其以后系统
* 2：设置的filter生效后，默认替代type和SubType


###  CAAnimationGrop

有时候，我们需要在对一个CALayer执行Position动画的同时，执行一个透明的opacity动画。简单的将两个动画行为添加到图层中，并不能完全的保证2个行为处于一个动画空间，或者是所受到的影响完全一样（比如同一时间的其他动画干扰）。CAAnimaitonGrop就是为了满足这个应用场景而设计的。

需要注意的是CAAnimaitonGrop不会改变其所包含的动画行为的时间。举个例子：

有一个CABAsicAnimaiton A, A.duraiton = 5s. 
有一个CAAnimaitonGrop  B, B包含A
当B.duration = 10s。 动画执行时间是10s,执行5s
当B.duration = 4s。  动画执行时间是4s,执行4s


### 总结

本章主要将了Core Animation提供的动画扩展功能

* CAAnimation是一个基本的抽象动画类，主要实现了动画的时间，动画获取通道，以后动画什么周期的代理回调

* CAPropertyAnimation是CAAnimation的一个抽象子类，主要实现了对CALayer的KeyPath变化的动画支持
* CABasicAnimation是CAPropertyAnimation的子类，主要实现对CALayer的单一属性变化的动画支持
* CAKeyframeAnimatio是CAPropertyAnimation的子类，主要提供了通过Path和Valuse两个接口控制动画关键帧的能力
* CATransition主要用于对整个CALayer的动画影响，除了苹果公开的动画类型，我们可以使用Fillter自定义转场动画。
* CAAnimationGroup主要是运行一堆动画在同时执行


### 参看文献

##### [controlling Animation Timing](http://ronnqvi.st/controlling-animation-timing/)

#### [Animation Types and Timing Programming Guide](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/Animation_Types_Timing/Introduction/Introduction.html)

#### [Key-Value Coding Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html)

