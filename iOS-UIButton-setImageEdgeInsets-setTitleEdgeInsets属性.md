
##  iOS UIButton setImageEdgeInsets setTitleEdgeInsets属性



使用UIButton的过程中不可避免的会遇到对Button中ImageView和TieleLabel布局修改的问题、官方也提供了：   

[button setImageEdgeInsets:UIEdgeInsets];   

[button setTitleEdgeInsets:UIEdgeInsets]   

这两个方法操作ImageView和TitleLabel的偏移量来达到各种各样的需求。

通常使用UIEdgeInsetsMake(top , left , bottom , right)；进行偏移量的设置   

top:在原坐标基础上向下移动   

left:在原坐标基础上向右移动   

bottom:在原坐标基础上向上移动   

right:在原坐标基础上向左移动   

以上表示设置参数为正时控件的移动方向、为负值移动方向相反。  

由于button对ImageView和TitleLabel同时存在时候的默认布局是ImageView在左TitleLabel在右、那么通过这个属性我们就可以满足类似以下需求 

1、图片在上文字在下  

2、图片在右文字在左

举个例子  

```
/*
ImageView在上、TitleLabel在下。
*注意获取titleLabel宽高用
button.titleLabel.intrinsicContentSize.height
button.titleLabel.intrinsicContentSize.width
获取
*/

[button setImageEdgeInsets:UIEdgeInsetsMake(-button.titleLabel.intrinsicContentSize.height, 0, 0, -button.titleLabel.intrinsicContentSize.width)];
        [button setTitleEdgeInsets:UIEdgeInsetsMake(0, -button.imageView.frame.size.width ,-button.imageView.frame.size.height, 0)];

/*
ImageView在右、TitleLabel在左。
*/

[button setTitleEdgeInsets:UIEdgeInsetsMake(0, 0, 0, button.imageView.image.size.width + 10)];
            [button setImageEdgeInsets:UIEdgeInsetsMake(0, 0, 0, -button.intrinsicContentSize.width+ 15)];
```



[返回首页](https://cwos111509sina.github.io/Blog/)
