---
title: 使用 unwind segues 关闭当前页面
date: 2018-04-25 11:59:33
tags:
	- iOS
---

我们经常会遇到这样一种需求，在控制器上有一个关闭按钮，点击来关闭当前页面。而我们通常的做法都是在 Storyboard 上拖一个 UIButton 出来，然后将 UIButton 拖线到控制器上，新建一个 IBAction 方法，并在该方法中通过代码来关闭当前控制器。

其实，我们可以不需要这么麻烦，可以通过 [`Unwind Segue`](https://developer.apple.com/library/content/technotes/tn2298/_index.html) 在 Storyboard 上拖拖线即可达到关闭控制器的效果。
 
## 创建一个 Unwind Segue

unwind是一个实例方法，以`UIStoryboardSegue`作为其唯一的方法参数，返回类型为`IBAction`，它对方法名称不作要求，也不需要在头文件中进行声明。

需要注意的是，我们需要将 uniwnd action 定义在 UIStoryboardSegue 的 sourceViewController 上，而非当前控制器。例如 A push B 然后从 B 返回 A 这个场景，我们需要将 unwind action 定义在 A 上。

但是我们可以通过扩展 UIViewController 以便所有控制器均可以使用，从而不用关心具体写在哪个控制器上。

```ObjC
- (IBAction)unwindSegue:(UIStoryboardSegue*)sender {
    // Pull any data from the view controller which initiated the unwind segue.
}
```

```Swift
extension UIViewController {
    
    @IBAction func unwindSegue(_ sender: UIStoryboardSegue) {
        // Pull any data from the view controller which initiated the unwind segue.
    }
    
}
```

## 将 Unwind Segue 添加到故事板

- 方法1

a. 选中退出图标，右击弹出 Segues 列表

![](/images/201804/52B0EF013.png)

b. 将 `unwindSegue:` 拖线到控制器图标上

![](/images/201804/70FA52B0EF01384A.png)

c. 设置一个 identifier

![](/images/201804/801A47A16506.png)

经过这样的拖线绑定之后，我们就可以在代码中通过发送一个 `performSegue(withIdentifier:sender:)` 消息来关闭当前控制器。


- 方法2

a. 选中按钮，然后按住 control 拖线到退出图标上

![](/images/201804/968888BC3109.png)

b. 在弹出的 Action Segues 中选择 `unwindSegue:`

![](/images/201804/F84E8854.png)

这样，当我们点击按钮的时候，就会自动关闭当前控制器了。


[Technical Note TN2298 Using Unwind Segues](https://developer.apple.com/library/content/technotes/tn2298/_index.html)
