 
 ## iOS 自定义表情键盘
 
 
直接上代码，需要自备表情包表情命名为”ee_1”–“ee_105” 表情按钮图片。后面有项目链接 

![](https://cwos111509sina.github.io/Blog/Source/OC/iOS-自定义表情键盘/emojikeyBoard_0.png)

```
#import "ViewController.h"

#define WIDTH [UIScreen mainScreen].bounds.size.width

#define HEIGHT [UIScreen mainScreen].bounds.size.height

#define WINDOW [UIApplication sharedApplication].keyWindow

#define kColor(x,y,z) [UIColor colorWithRed:x/255.0 green:y/255.0 blue:z/255.0 alpha:1]

@interface ViewController ()@property (nonatomic,strong)UILabel * textLab;

@property (nonatomic,strong)UIScrollView * scrollView;

@property (nonatomic,strong)UIPageControl * scrollPage;

@property (nonatomic,copy)NSMutableString * attriString;

@property (nonatomic,strong)UIView * editView;

@property (nonatomic,strong)UITextView * textView;

@end

@implementation ViewController

- (void)viewDidLoad {  

  [super viewDidLoad]; 

   self.view.backgroundColor = [UIColor whiteColor];   

    _attriString = [[NSMutableString alloc]init];        

[self createUI];        

}

-(void)createUI{            

_textLab = [[UILabel alloc]initWithFrame:CGRectMake(20, 20, WIDTH-40, 200)];    _textLab.font = [UIFont systemFontOfSize:15];    _textLab.layer.borderWidth = 2;    _textLab.layer.borderColor = [UIColor blackColor].CGColor;  

_textLab.textAlignment = NSTextAlignmentCenter;                

_editView = [[UIView alloc]initWithFrame:CGRectMake(0, HEIGHT-49, WIDTH, 49)];    _editView.backgroundColor = [UIColor whiteColor];       

 UIButton * button = [UIButton buttonWithType:UIButtonTypeCustom];    

button.frame = CGRectMake(10, 0, 49, 49);    

[button setImage:[UIImage imageNamed:@"expression"] forState:UIControlStateNormal];        

[button addTarget:self action:@selector(showEmojiView) forControlEvents:UIControlEventTouchUpInside];       

 [_editView addSubview:button];        

_textView = [[UITextView alloc]initWithFrame:CGRectMake(69, 5, WIDTH-129, 39)];    _textView.delegate = self;   

 _textView.backgroundColor = kColor(238, 238, 238);    

_textView.layer.cornerRadius = 3;    

_textView.clipsToBounds = YES;           

 [_editView addSubview:_textView];        

UIButton * sendButton = [UIButton buttonWithType:UIButtonTypeCustom];    sendButton.frame = CGRectMake(WIDTH-59, 0, 49, 49);        

[sendButton setTitle:@"发送" forState:UIControlStateNormal];    

[sendButton setTitleColor:[UIColor blueColor] forState:UIControlStateNormal];    [sendButton addTarget:self action:@selector(sendBtn) forControlEvents:UIControlEventTouchUpInside];        

[_editView addSubview:sendButton];                    

[self.view addSubview:_textLab];    

[self.view addSubview:_editView];                

UITapGestureRecognizer * tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(tapClick:)];   

 [self.view addGestureRecognizer:tap];    

}

#pragma mark 模仿发送按钮-解析表情展示到label

-(void)sendBtn{            

NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"%@",_attriString]];        

NSString * rugxString = @"\\[\\{e:([1-9]|[1-9][0-9]|[1][0-9][0-5])\\}\\]";        NSRegularExpression * re = [NSRegularExpression regularExpressionWithPattern:rugxString options:NSRegularExpressionCaseInsensitive error:nil];       

 NSArray * resultArray = [re matchesInString:_attriString options:0 range:NSMakeRange(0, _attriString.length)];        

NSMutableArray * RangeArray = [NSMutableArray arrayWithCapacity:resultArray.count];        

for (NSTextCheckingResult * result in resultArray) {                

NSRange range = [result range];                

NSString * string = [_attriString substringWithRange:range];                

for (int i = 1; i<106; i++) {                        

if ([[NSString stringWithFormat:@"[{e:%d}]",i] isEqualToString:string]) {                                UIImage * image = [UIImage imageNamed:[NSString stringWithFormat:@"ee_%d.png",i]];                

NSTextAttachment *imageAttachment = [[NSTextAttachment alloc]init];                imageAttachment.image = image;                                

imageAttachment.bounds = CGRectMake(0, 0,_textLab.font.lineHeight, _textLab.font.lineHeight);                                

NSAttributedString *imageAttributedString = [NSAttributedString attributedStringWithAttachment:imageAttachment];                                

NSDictionary * imgDict = @{@"image":imageAttributedString,@"range":[NSValue valueWithRange:range]};                                

[RangeArray addObject:imgDict];                           

 }                    

}    

}       

 for (int i = (int)RangeArray.count-1; i>=0; i--) {                

NSRange range;        

[RangeArray[i][@"range"] getValue:&range];                

[attributedString replaceCharactersInRange:range withAttributedString:RangeArray[i][@"image"]];            

}    

_textLab.attributedText = attributedString;        

}

#pragma mark 显示表情键盘

-(void)showEmojiView{       

 [_textView endEditing:YES];        

if (!_scrollView) {        

[self createExpressionView];   

 }    

[self changeEditViewFrameWithType:YES];        

_scrollView.hidden = NO;    

_scrollPage.hidden = NO;    

}

-(void)createExpressionView{   

 _scrollView = [[UIScrollView alloc]initWithFrame:CGRectMake(0, HEIGHT*860/1280, WIDTH, HEIGHT*420/1280-30)];    

_scrollView.backgroundColor = [UIColor whiteColor];    

_scrollView.contentSize = CGSizeMake(WIDTH*4,HEIGHT*420/1280-30);    _scrollView.showsHorizontalScrollIndicator = NO;    

_scrollView.pagingEnabled = YES;    

_scrollView.delegate = self;        

_scrollPage = [[UIPageControl alloc]initWithFrame:CGRectMake(0,CGRectGetMaxY(_scrollView.frame),WIDTH, 30)];    _scrollPage.backgroundColor = [UIColor whiteColor];    

_scrollPage.numberOfPages = 4;    

_scrollPage.pageIndicatorTintColor = [UIColor grayColor];    _scrollPage.currentPageIndicatorTintColor = kColor(64, 151, 222);    _scrollPage.currentPage = 0;        

[_scrollPage addTarget:self action:@selector(scrollPageClick:) forControlEvents:UIControlEventValueChanged];        

for (int i = 0; i<105; i++) {                

UIButton * emojiBtn = [UIButton buttonWithType:UIButtonTypeCustom];                emojiBtn.frame = CGRectMake(i%31%8*WIDTH/8+WIDTH*(int)(i/31), i%31%32/8*(HEIGHT*420/1280-30)/4, WIDTH/8, (HEIGHT*420/1280-30)/4);        emojiBtn.backgroundColor = [UIColor clearColor];       

 [emojiBtn setImage:[UIImage imageNamed:[NSString stringWithFormat:@"ee_%d",i+1]] forState:UIControlStateNormal];                

[emojiBtn setImageEdgeInsets:UIEdgeInsetsMake((emojiBtn.frame.size.height-HEIGHT*33/1280)/4, (WIDTH/8-WIDTH*33/720)/4, (emojiBtn.frame.size.height-HEIGHT*33/1280)/4, (WIDTH/8-WIDTH*33/720)/4)];                

emojiBtn.tag = 200+i;       

 [emojiBtn addTarget:self action:@selector(emojiBtnClick:) forControlEvents:UIControlEventTouchUpInside];       

 [_scrollView addSubview:emojiBtn];                                        

if ( i % 31 == 0) {            

UIButton * delButton = [UIButton buttonWithType:UIButtonTypeCustom];                        delButton.frame = CGRectMake(7*WIDTH/8+WIDTH*(int)(i/31), 3*(HEIGHT*420/1280-30)/4, WIDTH/8, (HEIGHT*420/1280-30)/4);            

delButton.backgroundColor = [UIColor clearColor];           

 [delButton setImage:[UIImage imageNamed:@"Expression_Del"] forState:UIControlStateNormal];            

[delButton addTarget:self action:@selector(delButtonClick:) forControlEvents:UIControlEventTouchUpInside];           

 [_scrollView addSubview:delButton];      

  }            

}           

 _scrollView.hidden = YES;    

_scrollPage.hidden = YES;    

[WINDOW addSubview:_scrollView];    

[WINDOW addSubview:_scrollPage];

}

#pragma mark        emojiBtnClick按钮

-(void)emojiBtnClick:(UIButton *)emojiBtn{        

[_attriString appendFormat:@"[{e:%d}]",(int)emojiBtn.tag-199];        NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:_attriString];    

NSString * rugxString = @"\\[\\{e:([1-9]|[1-9][0-9]|[1][0-9][0-5])\\}\\]";    NSRegularExpression * re = [NSRegularExpression regularExpressionWithPattern:rugxString options:NSRegularExpressionCaseInsensitive error:nil];    

NSArray * resultArray = [re matchesInString:_attriString options:0 range:NSMakeRange(0, _attriString.length)];    

NSMutableArray * RangeArray = [NSMutableArray arrayWithCapacity:resultArray.count];        

for (NSTextCheckingResult * result in resultArray) {        

NSRange range = [result range];       

 NSString * string = [_attriString substringWithRange:range];        

for (int i = 1; i<106; i++) {            

if ([[NSString stringWithFormat:@"[{e:%d}]",i] isEqualToString:string]) {                                UIImage * image = [UIImage imageNamed:[NSString stringWithFormat:@"ee_%d.png",i]];                

NSTextAttachment *imageAttachment = [[NSTextAttachment alloc]init];                imageAttachment.image = image;                

imageAttachment.bounds = CGRectMake(0, 0,17.9, 17.9);                

NSAttributedString *imageAttributedString = [NSAttributedString attributedStringWithAttachment:imageAttachment];                

NSDictionary * imgDict = @{@"image":imageAttributedString,@"range":[NSValue valueWithRange:range]};                

[RangeArray addObject:imgDict];                           

 }        

}    

}   

 [attributedString setAttributes:@{NSFontAttributeName:[UIFont systemFontOfSize:15]} range:NSMakeRange(0, attributedString.length)];        

for (int i = (int)RangeArray.count-1; i>=0; i--) {                

NSRange range;       

 [RangeArray[i][@"range"] getValue:&range];               

 [attributedString replaceCharactersInRange:range withAttributedString:RangeArray[i][@"image"]];    }        

_textView.attributedText = attributedString;    

}

#pragma mark        delButtonClick按钮

-(void)delButtonClick:(UIButton *)delBtn{        

NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:_attriString];    

NSString * rugxString = @"\\[\\{e:([1-9]|[1-9][0-9]|[1][0-9][0-5])\\}\\]";    NSRegularExpression * re = [NSRegularExpression regularExpressionWithPattern:rugxString options:NSRegularExpressionCaseInsensitive error:nil];    

NSArray * resultArray = [re matchesInString:_attriString options:0 range:NSMakeRange(0, _attriString.length)];    

NSMutableArray * RangeArray = [NSMutableArray arrayWithCapacity:resultArray.count];        

for (NSTextCheckingResult * result in resultArray) {        

NSRange range = [result range];        

NSString * string = [_attriString substringWithRange:range];        

for (int i = 1; i<106; i++) {            

if ([[NSString stringWithFormat:@"[{e:%d}]",i] isEqualToString:string]) {                                UIImage * image = [UIImage imageNamed:[NSString stringWithFormat:@"ee_%d.png",i]];                

NSTextAttachment *imageAttachment = [[NSTextAttachment alloc]init];                imageAttachment.image = image;               

 imageAttachment.bounds = CGRectMake(0, 0,17.9, 17.9);                

NSAttributedString *imageAttributedString = [NSAttributedString attributedStringWithAttachment:imageAttachment];                

NSDictionary * imgDict = @{@"image":imageAttributedString,@"range":[NSValue valueWithRange:range]};               

 [RangeArray addObject:imgDict]; 

                           }        

}    

}   

 [attributedString setAttributes:@{NSFontAttributeName:[UIFont systemFontOfSize:15]} range:NSMakeRange(0, attributedString.length)];        

for (int i = (int)RangeArray.count-1; i>=0; i--) {               

 NSRange range;       

 [RangeArray[i][@"range"] getValue:&range];        

if (i == RangeArray.count-1) {            

[attributedString deleteCharactersInRange:range];           

 [_attriString replaceCharactersInRange:range withString:@""];        

}else{            

[attributedString replaceCharactersInRange:range withAttributedString:RangeArray[i][@"image"]];       

 }   

 }   

 _textView.attributedText = attributedString;    

}

#pragma  mark scrollViewPage按钮

-(void)scrollPageClick:(UIPageControl *)scrollPage{        

[_scrollView setContentOffset:CGPointMake(WIDTH*scrollPage.currentPage, 0)];    }

#pragma mark scrollViewDelegate

-(void)scrollViewDidScroll:(UIScrollView *)scrollView{    

_scrollPage.currentPage = scrollView.contentOffset.x/WIDTH;

}

#pragma mark textviewDelegate

-(void)textViewDidBeginEditing:(UITextView *)textView{       

 [self changeEditViewFrameWithType:YES];        

_scrollPage.hidden = YES;    

_scrollView.hidden = YES;    

}

-(void)textViewDidChange:(UITextView *)textView{        

if ([_attriString isEqualToString:textView.text]) {    

}else{ 

               NSString * rugxString = @"\\[\\{e:([1-9]|[1-9][0-9]|[1][0-9][0-5])\\}\\]";        NSRegularExpression * re = [NSRegularExpression regularExpressionWithPattern:rugxString options:NSRegularExpressionCaseInsensitive error:nil];       

 NSArray * resultArray = [re matchesInString:_attriString options:0 range:NSMakeRange(0, _attriString.length)];                

NSMutableArray * RangeArray = [NSMutableArray arrayWithCapacity:resultArray.count];        

for (NSTextCheckingResult * result in resultArray) {           

 NSRange AttRange = [result range];            

NSString * string = [_attriString substringWithRange:AttRange];            

for (int i = 1; i<106; i++) {                

if ([[NSString stringWithFormat:@"[{e:%d}]",i] isEqualToString:string]) {                    [RangeArray addObject:[NSValue valueWithRange:AttRange]];                

}           

 }       

 }               

 NSMutableArray * textArr = [[NSMutableArray alloc]init];        

for (int i = (int)RangeArray.count-1; i>=0; i--) {            

NSRange AttRange;            

[RangeArray[i] getValue:&AttRange];           

 [textArr addObject:[_attriString substringWithRange:AttRange]];         

   [_attriString replaceCharactersInRange:AttRange withString:@" "];          

  for (int y = 0; y= 0 ; i--) {                    

    NSRange attRange;             

           [RangeArray[i] getValue:&attRange];          

              [_attriString replaceCharactersInRange:attRange withString:textArr[y]];                        y++;        }            }    }

-(BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text{      

  NSString * rugxString = @"\\[\\{e:([1-9]|[1-9][0-9]|[1][0-9][0-5])\\}\\]";    NSRegularExpression * re = [NSRegularExpression regularExpressionWithPattern:rugxString options:NSRegularExpressionCaseInsensitive error:nil]; 

   NSArray * resultArray = [re matchesInString:_attriString options:0 range:NSMakeRange(0, _attriString.length)];    

    NSMutableArray * RangeArray = [NSMutableArray arrayWithCapacity:resultArray.count];   

 for (NSTextCheckingResult * result in resultArray) {     

   NSRange AttRange = [result range];       

 NSString * string = [_attriString substringWithRange:AttRange];   

     for (int i = 1; i<106; i++) {          

  if ([[NSString stringWithFormat:@"[{e:%d}]",i] isEqualToString:string]) {                [RangeArray addObject:[NSValue valueWithRange:AttRange]];    

        }    

    }   

 }  

  NSMutableArray * textArr = [[NSMutableArray alloc]init];    

for (int i = (int)RangeArray.count-1; i>=0; i--) {      

  NSRange AttRange;     

   [RangeArray[i] getValue:&AttRange];     

   [textArr addObject:[_attriString substringWithRange:AttRange]];   

     [_attriString replaceCharactersInRange:AttRange withString:@" "];        AttRange.length = 1;     

   for (int y = 0; y= 0 ; i--) {

NSRange attRange;

[RangeArray[i] getValue:&attRange];

if (attRange.location>range.location) {

attRange.location += range.length;

}

if (range.location == attRange.location && range.length == attRange.length) {

}else{

[_attriString replaceCharactersInRange:attRange withString:textArr[y]];

}

y++;

}

return YES;

}

-(void)tapClick:(UITapGestureRecognizer *)tap{

[_textView endEditing:YES];

[self changeEditViewFrameWithType:NO];

_scrollView.hidden = YES;

_scrollPage.hidden = YES;

}

-(void)changeEditViewFrameWithType:(BOOL)up{

if (up) {

_editView.frame = CGRectMake(0, HEIGHT - 256 - 49 , WIDTH, 49);

}else{

_editView.frame = CGRectMake(0, HEIGHT - 49 , WIDTH, 49);

}

}

@end
```

[项目地址](https://github.com/cwos111509sina/EmojiKeyBoard.git)

 [返回首页](https://cwos111509sina.github.io/Blog/)
