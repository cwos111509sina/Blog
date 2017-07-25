
##  iOS 扫描二维码、长按图片识别二维码


昨天自定义表情键盘的时候发现写一些东西是必要的，不只是以后方便自己查找，重要的是再一次梳理的时候会加深印象，而且会把当时不清不楚的东西重新拿出来思考一遍，温故知新吧。作为一个菜鸟，尝试写了个二维码扫面和长按识别。下面有链接：   
其中用到的内容包括：

info.plist 设置Privacy - Camera Usage Description - 是否允许此App使用你的相机？

AVFoundation库

AVCaptureDevice 设备

AVCaptureDeviceInput 输入

AVCaptureMetadataOutput 输出

AVCaptureSession 链接输入及输出内容

AVCaptureVideoPreviewLayer 扫描预览layer

AVCaptureMetadataOutputObjectsDelegate 扫面代理方法查看扫描结果

CIDetector 长按识别扫描仪 
CIQRCodeFeature 获取识别结果

代码：

```
import “ViewController.h” 
import AVFoundation/AVFoundation.h

define WIDTH [UIScreen mainScreen].bounds.size.width

define HEIGHT [UIScreen mainScreen].bounds.size.height


@interface ViewController ()

@property (nonatomic,strong)AVCaptureSession * session;

@property (nonatomic,strong)UIView * scansoinView;

@property (nonatomic,strong)UIImageView * distinguishView;

@property (nonatomic,strong)UILabel * resultLab;

@end

@implementation ViewController

(void)viewDidLoad { 
[super viewDidLoad];

self.view.backgroundColor = [UIColor whiteColor];

[self createUI];

// Do any additional setup after loading the view, typically from a nib. 
}

-(void)createUI{

_resultLab = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, WIDTH, 64)];
_resultLab.backgroundColor = [UIColor blackColor];

_resultLab.text = @"扫描结果";//
_resultLab.textAlignment = NSTextAlignmentCenter;

_resultLab.textColor = [UIColor whiteColor];

_resultLab.numberOfLines = 0;

[self.view addSubview:_resultLab];


[self createScansionView];

[self createDistinguishView];


NSArray * array = @[@"扫描二维码",@"长按图片识别二维码"];

for (int i = 0; i<array.count; i++) {

    UIButton * button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    button.frame = CGRectMake(30, HEIGHT-100 + i*50, WIDTH-60, 40);


    [button setTitle:array[i] forState:UIControlStateNormal];

    button.backgroundColor = [UIColor blackColor];

    [button setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];

    button.tag = 100+i;

    [button addTarget:self action:@selector(buttonClick:) forControlEvents:UIControlEventTouchUpInside];


    [self.view addSubview:button];

}
} 
pragma mark ——————创建扫描页面 
-(void)createScansionView{

NSString *mediaType = AVMediaTypeVideo;

AVAuthorizationStatus authStatus = [AVCaptureDevice authorizationStatusForMediaType:mediaType];

if(authStatus == AVAuthorizationStatusRestricted || authStatus == AVAuthorizationStatusDenied){

    UIAlertView *alert =[[UIAlertView alloc]initWithTitle:@"获取权限" message:@"请在iPhone的“设置”-“隐私”-“相机”功能中，找到“XXXX”打开相机访问权限" delegate:nil cancelButtonTitle:@"确定" otherButtonTitles: nil];

    [alert show];

    return;

}
_scansoinView = [[UIView alloc]initWithFrame:CGRectMake(0, 64, WIDTH, HEIGHT-184)];

_scansoinView.backgroundColor = [UIColor blackColor];

AVCaptureDevice * captureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
AVCaptureDeviceInput * deviceInput = [AVCaptureDeviceInput deviceInputWithDevice:captureDevice error:nil];
AVCaptureMetadataOutput * output = [[AVCaptureMetadataOutput alloc]init];

[output setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];

_session = [[AVCaptureSession alloc]init];
[_session setSessionPreset:AVCaptureSessionPresetHigh];

if ([_session canAddInput:deviceInput]) {
    [_session addInput:deviceInput];
}
if ([_session canAddOutput:output]) {
    [_session addOutput:output];
}

output.metadataObjectTypes = @[AVMetadataObjectTypeQRCode];


AVCaptureVideoPreviewLayer * previewLayer = [AVCaptureVideoPreviewLayer layerWithSession:_session];
previewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;

previewLayer.frame = _scansoinView.layer.bounds;

[_scansoinView.layer insertSublayer:previewLayer atIndex:0];

UILabel * xianLab = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, WIDTH, 1)];
xianLab.backgroundColor = [UIColor redColor];

[_scansoinView addSubview:xianLab];

[UIView animateWithDuration:2.5 delay:0.0 options:UIViewAnimationOptionRepeat animations:^{
    xianLab.frame = CGRectMake(0, HEIGHT-184, WIDTH, 1);
} completion:nil];



[self.view addSubview:_scansoinView];

[_session startRunning];
}

pragma mark ——————创建识别页面 
-(void)createDistinguishView{

_distinguishView = [[UIImageView alloc]initWithFrame:CGRectMake(0, 64, WIDTH, HEIGHT-184)];

_distinguishView.image = [UIImage imageNamed:@"QRCodePic"];

_distinguishView.userInteractionEnabled = YES;

UILongPressGestureRecognizer * longPress = [[UILongPressGestureRecognizer alloc]initWithTarget:self action:@selector(distinguishClick:)];

[_distinguishView addGestureRecognizer:longPress];

_distinguishView.hidden = YES;
[self.view addSubview:_distinguishView];
}

-(void)distinguishClick:(UILongPressGestureRecognizer *)longPress{

UIImageView * imageV = (UIImageView *)longPress.view;

//1. 初始化扫描仪，设置设别类型和识别质量
CIDetector*detector = [CIDetector detectorOfType:CIDetectorTypeQRCode context:nil options:@{ CIDetectorAccuracy : CIDetectorAccuracyHigh }];
//2. 扫描获取的特征组
NSArray *features = [detector featuresInImage:[CIImage imageWithCGImage:imageV.image.CGImage]];
//3. 获取扫描结果
CIQRCodeFeature *feature = [features objectAtIndex:0];

_resultLab.text = [NSString stringWithFormat:@"长按识别二维码结果:%@",feature.messageString];

NSLog(@"长按识别二维码结果：%@",feature.messageString);
}

-(void)buttonClick:(UIButton *)btn{

if (btn.tag == 100) {//扫描二维码

    if (!_scansoinView.isHidden) {
        return;
    }

    _scansoinView.hidden = NO;
    _distinguishView.hidden = YES;
    [_session startRunning];

}else{//识别二维码

    if (!_distinguishView.isHidden) {
        return;
    }
    [_session stopRunning];

    _scansoinView.hidden = YES;
    _distinguishView.hidden = NO;

}
}

-(void)captureOutput:(AVCaptureOutput )captureOutput didOutputMetadataObjects:(NSArray )metadataObjects fromConnection:(AVCaptureConnection *)connection{

[_session stopRunning];

AVMetadataMachineReadableCodeObject * codeOBJ = metadataObjects[0];


_resultLab.text = [NSString stringWithFormat:@"扫描结果:%@",codeOBJ.stringValue];

NSLog(@"扫描结果：metadataObjects = %@",codeOBJ.stringValue);
}

@end 
```

[项目链接](https://github.com/cwos111509sina/QRCodeText.git)

[返回首页](https://cwos111509sina.github.io/Blog/)
