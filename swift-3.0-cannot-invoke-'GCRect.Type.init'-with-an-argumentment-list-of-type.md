
##  swift 3.0 - cannot invoke 'GCRect.Type.init' with an argumentment list of type

刚接触Swift，创建button设置frame的时候爆了这样一个问题，最后把自定义宽高数据类型转换为Int类型的时候解决 


```
//错误

for index in (0...titleArr.count-1){

  let button = UIButton.init(type: .custom)
  
  button.frame = CGRect.init(x: imgW, y: imgH, x: imgWidth, x: imgWidth)//报错：cannot invoke 'GCRect.Type.init' with an argumentment list of type
  
}

//正确

var imgW : Int = 1;
var imgH : Int = 216;

let imgWidth : Int = (Int)(WIDTH-4)/3

for index in (0...titleArr.count-1){
  
  let button = UIButton.init(type: .custom)
  
  button.frame = CGRect.init(x: imgW, y: imgH, x: imgWidth, x: imgWidth)
  
}

```



[返回首页](https://cwos111509sina.github.io/Blog/)
