
## 自定义时间选择器

提供6种时间选择方式：
年月日、年月日时、年月日时分、时分、日时分、月日时分

### 调用方式
```

 [[DatePicker shareManager]showWithType:PickerTypeDay title:nil time:@"2018:10:31" backTime:^(NSString * _Nonnull backTimeStr) {
        NSLog(@"%@",backTimeStr);
    }];

```

### DatePicker.h
```

typedef NS_ENUM (NSInteger ,PickerType){

    PickerTypeDay = 3,//年月日
    PickerTypeHour,//年月日时
    PickerTypeMinute,//年月日时分
    PickerTypeMinuteHour,//时分
    PickerTypeMinuteDay,//日时分
    PickerTypeMinuteMonth,//月日时分
};

@interface DatePicker : UIView

+(instancetype)shareManager;
-(instancetype)init;

/*
 
 pickerType 时间选择器类型
 title      标题（可不传）
 time       传入的值会在选择器中选中展示（可不传）
 backTime   返回时间（例：yyyy:MM:dd:HH:mm）
 */
-(void)showWithType:(PickerType)pickerType title:(NSString * _Nullable)title time:(NSString * _Nullable )timeStr backTime:(void (^)(NSString * backTimeStr))backTime;

@end

```

### DatePicker.m

```

#import "DatePicker.h"


#define WIDTH [UIScreen mainScreen].bounds.size.width
#define HEIGHT [UIScreen mainScreen].bounds.size.height
#define KEYWINDOW [UIApplication sharedApplication].keyWindow



@interface DatePicker ()<UIPickerViewDelegate,UIPickerViewDataSource>


@property (nonatomic,strong)UIView * bgView;//背景
@property (nonatomic,strong)UILabel * titlelabel;//标题
@property (nonatomic,strong)UIPickerView * pickerView;//选择器

@property (nonatomic,assign)PickerType pickerType;//类型

@property (nonatomic,strong)NSDateComponents * components;//时间结构体、可计算时间间隔、获取年月日时分、
@property (nonatomic,strong)NSMutableArray * dataArray;//数据源
@property (nonatomic,strong)NSMutableArray * selDateArray;
@property (nonatomic,strong)void (^backTime)(NSString * backTimeStr);//回调

@end


@implementation DatePicker

+(instancetype)shareManager{
    
    static DatePicker * manager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (!manager) {
            manager = [[DatePicker alloc] init];
        }
    });
    return manager;
}
-(instancetype)init{
    if (self = [super init]) {
        [self createView];
    }
    return self;
}

-(void)createView{
    
    self.frame = CGRectMake(0, 0, WIDTH, HEIGHT);
    self.backgroundColor = [[UIColor blackColor] colorWithAlphaComponent:0.3];
    _dataArray = [[NSMutableArray alloc]init];
    
    _bgView = [[UIView alloc]initWithFrame:CGRectMake((WIDTH-250)/2, HEIGHT/2-124, 250, 200)];
    _bgView.backgroundColor = [UIColor whiteColor];
    _bgView.layer.cornerRadius = 5;
    
    _titlelabel = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, _bgView.bounds.size.width, 33)];
    _titlelabel.textAlignment = NSTextAlignmentCenter;
    [_bgView addSubview:_titlelabel];
    
    
    _pickerView = [[UIPickerView alloc]initWithFrame:CGRectMake(0, CGRectGetMaxY(_titlelabel.frame), _bgView.bounds.size.width, _bgView.frame.size.height-CGRectGetMaxY(_titlelabel.frame)-33)];
    _pickerView.delegate = self;
    _pickerView.dataSource = self;
    [_bgView addSubview:_pickerView];
    
    
    UIButton * cancelButton = [UIButton buttonWithType:UIButtonTypeCustom];
    cancelButton.frame = CGRectMake(0, CGRectGetMaxY(_pickerView.frame), _bgView.bounds.size.width/2-0.5, 33);
    [cancelButton setTitle:@"取消" forState:UIControlStateNormal];
    [cancelButton addTarget:self action:@selector(cancelButtonClick:) forControlEvents:UIControlEventTouchUpInside];
    [cancelButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [_bgView addSubview:cancelButton];
    
    
    UIButton * trueButton = [UIButton buttonWithType:UIButtonTypeCustom];
    trueButton.frame = CGRectMake(_bgView.bounds.size.width/2+0.5, CGRectGetMaxY(_pickerView.frame), _bgView.bounds.size.width/2, 33);
    [trueButton setTitle:@"确定" forState:UIControlStateNormal];
    [trueButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [trueButton addTarget:self action:@selector(trueButtonClick:) forControlEvents:UIControlEventTouchUpInside];
    [_bgView addSubview:trueButton];
    
    [self addSubview:_bgView];
}


-(void)showWithType:(PickerType)pickerType title:(NSString * _Nullable)title time:( NSString * _Nullable )timeStr backTime:(void (^)(NSString * backTimeStr))backTime{
    
    _backTime = backTime;
    _pickerType = pickerType;
    _titlelabel.text = title;
    _selDateArray = [[NSMutableArray alloc]init];

    NSCalendar * calender = [NSCalendar currentCalendar];
    _components = [calender components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay|NSCalendarUnitHour|NSCalendarUnitMinute fromDate:[NSDate date]];
    
    NSArray * timeArr = [[NSArray alloc]init];
    if (timeStr) {
        timeArr = [self receiveTime:timeStr];
    }
    NSInteger count = (_pickerType>5)?_pickerType-4:_pickerType;
    if (timeArr.count == count) {//需要显示传入的时间
        
        _dataArray = [self dataSourceArray];
        int index = (_pickerType<6)?2:((_pickerType == 8)?1:0);
        if (index && timeArr.count == count) {
            NSInteger year = (index == 1)?_components.year:[timeArr[index-2] integerValue];
            [_dataArray replaceObjectAtIndex:index withObject:[self dayArrayMonth:[timeArr[index-1] integerValue] year:year]];
        }
        [_pickerView reloadAllComponents];
        
        for (int i = 0; i<count; i++) {
            [pickerView selectRow:[_dataArray[i] containsObject:timeArr[i]]?[_dataArray[i] indexOfObject:timeArr[i]]:0 inComponent:i animated:YES];
        }
        [_selDateArray addObjectsFromArray:timeArr];
    }else{//不需要显示传入的时间
        _dataArray = [self dataSourceArray];
        [_pickerView reloadAllComponents];
        
        
        NSArray * dateArr = @[[NSString stringWithFormat:@"%ld",_components.year],[NSString stringWithFormat:@"%ld",_components.month],[NSString stringWithFormat:@"%ld",_components.day],[NSString stringWithFormat:@"%ld",_components.hour],[NSString stringWithFormat:@"%ld",_components.minute]];

        if (_pickerType>5) {
            NSArray * arary = @[[NSNumber numberWithInteger:0],[NSNumber numberWithInteger:5],[NSNumber numberWithInteger:0],[NSNumber numberWithInteger:0]];
            for (int i = 0; i<_pickerType - 4; i++) {
                [_pickerView selectRow:[arary[i] integerValue] inComponent:_pickerType - 5 - i animated:YES];
                [_selDateArray insertObject:dateArr[4-i] atIndex:i];
            }
        }else{
            NSArray * arary = @[[NSNumber numberWithInteger:_components.month-1],[NSNumber numberWithInteger:_components.day-1],[NSNumber numberWithInteger:_components.hour],[NSNumber numberWithInteger:_components.minute]];
            for (int i = 1; i<_pickerType; i++) {
                [_pickerView selectRow:[arary[i] integerValue] inComponent:i animated:YES];
                [_selDateArray addObject:dateArr[i]];
            }
        }
        
    }
    
    //选中行边框颜色
    
    if (_pickerView.subviews.count>0) {
        [_pickerView.subviews objectAtIndex:1].layer.borderWidth = 0.5f;
        [_pickerView.subviews objectAtIndex:2].layer.borderWidth = 0.5f;
        [_pickerView.subviews objectAtIndex:1].layer.borderColor = [UIColor blackColor].CGColor;
        [_pickerView.subviews objectAtIndex:2].layer.borderColor = [UIColor blackColor].CGColor;
    }
    [KEYWINDOW addSubview:self];
}

-(NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView{
    return _pickerType<6?_pickerType:_pickerType-4;
}

-(NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component{
    return [_dataArray[component] count];
}

-(CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component{
    return (pickerView.bounds.size.width-10)/(_pickerType<6?_pickerType:_pickerType-4);
}
-(CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component{
    return pickerView.bounds.size.height/3;
}

-(UIView *)pickerView:(UIPickerView *)pickerView viewForRow:(NSInteger)row forComponent:(NSInteger)component reusingView:(UIView *)view{
    
     UILabel * lab = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, (pickerView.bounds.size.width-10)/(_pickerType<6?_pickerType:(_pickerType-4)), pickerView.frame.size.height/3)];
    lab.font = [UIFont systemFontOfSize:14];
    lab.textAlignment = NSTextAlignmentCenter;
    
    if (_selDateArray.count == _dataArray.count) {

        if (row == [pickerView selectedRowInComponent:component]) {
            NSMutableArray * array = [[NSMutableArray alloc]initWithArray:@[@"年",@"月",@"日",@"时",@"分"]];
        
            if (_pickerType>5) {
                for (int i = 0; i<9-_pickerType; i++) {
                    [array removeObjectAtIndex:0];
                }
            }
        
            NSString * str = [NSString stringWithFormat:@"%@%@",_dataArray[component][row],array[component]];
            NSMutableAttributedString * attrubutedStr = [[NSMutableAttributedString alloc]initWithString:str];
        
            [attrubutedStr setAttributes:@{NSForegroundColorAttributeName:[UIColor lightGrayColor]} range:NSMakeRange([str rangeOfString:array[component]].location, 1)];
        
            lab.textColor = [UIColor redColor];
            lab.attributedText = attrubutedStr;
        
        }else{
            lab.text = _dataArray[component][row];
            lab.textColor = [UIColor blackColor];
        
        }
    }else{
        lab.text = _dataArray[component][row];
        lab.textColor = [UIColor blackColor];
        
    }
    
    return lab;
}

-(void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component{
    
    [pickerView reloadComponent:component];
    int index = (_pickerType<6 && component == 1)?2:((_pickerType == 8 && component == 0)?1:0);

    if (index) {//选中月份时刷新天数
        NSInteger year = (index == 1)?_components.year:[_selDateArray[index-2] integerValue];
        [_dataArray replaceObjectAtIndex:index withObject:[self dayArrayMonth:[_dataArray[component][row] integerValue] year:year]];
        
        NSInteger selRow = [pickerView selectedRowInComponent:index];
        [pickerView reloadComponent:index];
        
        if (selRow<[_dataArray[index] count]) {
        }else{
            [pickerView selectRow:0 inComponent:index animated:YES];
            [_selDateArray replaceObjectAtIndex:index withObject:_dataArray[index][0]];
        }
    }
    
    [_selDateArray replaceObjectAtIndex:component withObject:_dataArray[component][row]];
}
//确定
-(void)trueButtonClick:(UIButton *)button{
    self.backTime([_selDateArray componentsJoinedByString:@":"]);
    [self removeFromSuperview];
}
//取消
-(void)cancelButtonClick:(UIButton *)button{
    [self removeFromSuperview];
}

//数据源
-(NSMutableArray *)dataSourceArray{
    NSMutableArray * sourceArray = [[NSMutableArray alloc]init];
    NSArray * yearArr = @[[NSString stringWithFormat:@"%ld",_components.year],
                          [NSString stringWithFormat:@"%ld",_components.year+1],
                          [NSString stringWithFormat:@"%ld",_components.year+2],
                          [NSString stringWithFormat:@"%ld",_components.year+3],
                          [NSString stringWithFormat:@"%ld",_components.year+4]];
    NSArray * monthArr = @[@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12"];
    NSArray * dayArr = [self dayArrayMonth:_components.month year:_components.year];
    NSArray * hourArr = @[@"00",@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12",@"13",@"14",@"15",@"16",@"17",@"18",@"19",@"20",@"21",@"22",@"23"];
    
    NSMutableArray * minutesArr = [[NSMutableArray alloc]init];
    for (int i = 0 ; i<60; i++) {
        [minutesArr addObject:[NSString stringWithFormat:@"%@%d",i<10?@"0":@"",i]];
    }
    NSArray * array = @[yearArr,monthArr,dayArr,hourArr,minutesArr];

    switch (_pickerType) {
        case PickerTypeDay: case PickerTypeHour: case PickerTypeMinute:{//年月日 & 时 & 分
            for (int i = 0; i<_pickerType; i++) {
                [sourceArray addObject:array[i]];
            }
        }
            break;
        case PickerTypeMinuteHour: case PickerTypeMinuteDay: case PickerTypeMinuteMonth:{//月 & 日 & 时分
            for (int i = 0; i<_pickerType-4; i++) {
                [sourceArray insertObject:array[4-i] atIndex:0];
            }
        }
            break;
        default:
            break;
    }
    
    return sourceArray;
}

//根据月份返回日期
-(NSArray *)dayArrayMonth:(NSInteger)month year:(NSInteger)year{
    NSArray * dayArr = [[NSArray alloc]init];
    
    switch (month) {
        case 1: case 3: case 5: case 7: case 8: case 10: case 12:
            dayArr = @[@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12",@"13",@"14",@"15",@"16",@"17",@"18",@"19",@"20",@"21",@"22",@"23",@"24",@"25",@"26",@"27",@"28",@"29",@"30",@"31"];
            
            break;
        case 4: case 6: case 9: case 11:
            dayArr = @[@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12",@"13",@"14",@"15",@"16",@"17",@"18",@"19",@"20",@"21",@"22",@"23",@"24",@"25",@"26",@"27",@"28",@"29",@"30"];
            break;
        case 2:
            if (year%4) {
                dayArr = @[@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12",@"13",@"14",@"15",@"16",@"17",@"18",@"19",@"20",@"21",@"22",@"23",@"24",@"25",@"26",@"27",@"28"];
            }else{
                dayArr = @[@"01",@"02",@"03",@"04",@"05",@"06",@"07",@"08",@"09",@"10",@"11",@"12",@"13",@"14",@"15",@"16",@"17",@"18",@"19",@"20",@"21",@"22",@"23",@"24",@"25",@"26",@"27",@"28",@"29"];
                
            }
            break;
            
        default:
            break;
    }
    return dayArr;
    
}
//分解传来需要显示的时间
-(NSArray *)receiveTime:(NSString *)timeStr{
    
    NSArray * timeArray = [timeStr componentsSeparatedByString:@":"];
    NSMutableArray * timeArr = [[NSMutableArray alloc]init];
    for (int i = 0; i<timeArray.count; i++) {
        NSString * str = [timeArray objectAtIndex:i];
        if (str.length == 1) {
            [timeArr addObject:[NSString stringWithFormat:@"0%@",str]];
        }else if(str.length == 0){
            [timeArr addObject:@"00"];
        }else{
            [timeArr addObject:str];
        }
    }
    
    return timeArr;
}


@end

```

[项目地址](https://github.com/cwos111509sina/DatePicker.git)  
[返回首页](https://cwos111509sina.github.io/Blog/)


