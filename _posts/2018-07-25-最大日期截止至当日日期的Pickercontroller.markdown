---
layout: post
title:  "最大日期截止至当日日期的Pickercontroller"
date:   2018-07-25 17:07:50 +0800
categories: jekyll update
---
可选最大日期至当日日期的日期轮选器

PickerController 在我们日常开发中算是十分常见的了，例如 省市区地理选择、某些分类选择 以及 本案例的日期选择 等等。

其中苹果官方特意为了方便广大开发者而单独出了一个 UIDataPicker 来用作日期时间选择的轮选器。

* 他直接内部封装好了 年、月、日 时间分组，并且完全按照标准万年历来划定时间，免去了我们手动分组分类计算的麻烦。（无非就是 大月小月平月闰月 的计算，但现成的总比现写的强，还包正确）

* 提供 minData、maxData 属性来设置选择范围。

这两条优势下来基本上来说就和写一个简单的 UILabel 一样了，免去我们很多不必要的麻烦更不要说还有些其他的定义。

但同样的，虽然书写灵活，但面对复杂多变的应用环境，他仍无法做到随心所欲的适用各场景。

例如本案例的要求：用户生日日期选择最大只可选择当日日期。

也许有的朋友会说：那就实时改变 maxData 就好了嘛！

话虽如此，但 UIDataPicker 是动画停止选中后如果超出 maxData 才自动回滚到 maxData 的 年、月、日。

也就是说，如果用户闲（e）来（yi）无（cao）趣（zuo）的话，就会出现：生命不息，动画不止 的无限滚动下去。另外 UIDataPicker 的设定是，minData、maxData 范围之外的日期也正常展示，只不过是有一个至灰提醒罢了。这样一来的话一方面可能造成用户体验不好，另一方面也可能不符合产品本身的目的。（OS：赶紧选完生日用我们的 app ，不要这这里瞎玩浪费我们的时间！）

基于上述原因，用回 PickerController 是十分正确的选择。

思路及部分代码展示：
```
// 本项目范围：1950-01-01 —— 当日日期
{
    // 选中的 年、月、日 在其对应数据源中的位置
    NSInteger yearIndex;
    NSInteger monthIndex;
    NSInteger dayIndex;
}
@property (nonatomic, strong) NSDateComponents *comp; // 当日日期 暨日期选择上限，如我在本帖发布日使用则为 2018-07-25
@property (nonatomic, strong) NSMutableArray *yearArray; //
@property (nonatomic, strong) NSMutableArray *monthArray;
@property (nonatomic, strong) NSMutableArray *dayArray;
```
初始化时获取当日日期：
```
NSCalendar *calendar = [[NSCalendar alloc]
                        initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
// 定义一个时间字段的旗标，指定将会获取指定年、月、日、时、分、秒的信息
unsigned unitFlags = NSCalendarUnitYear | NSCalendarUnitMonth |  NSCalendarUnitDay | NSCalendarUnitHour |  NSCalendarUnitMinute | NSCalendarUnitSecond | NSCalendarUnitWeekday;
// 获取不同时间字段的信息
self.comp = [calendar components: unitFlags fromDate:[NSDate date]];

// NSDateComponents 这个类十分简单，年、月、日 被定义成了属性，只需要 .year、.month、.day 就可以了，又兴趣的可以去看一下
yearIndex = [self.yearArray indexOfObject:[NSString stringWithFormat:@"%ld年", self.comp.year]];
monthIndex = [self.monthArray indexOfObject:[NSString stringWithFormat:@"%02ld月", self.comp.month]];
dayIndex = [self.dayArray indexOfObject:[NSString stringWithFormat:@"%02ld日", self.comp.day]];
```

UIPickerView DataSource （部分）
```
// 较正常的代码逻辑而言这里多了个判断当日日期时 月、日 的数组信息展示
- (NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component {
    if (component == 0) {
        return self.yearArray.count;
    } else if(component == 1) {
        // 如果是今年返回当前已过月分数
        if ([self.yearArray[yearIndex] isEqualToString:self.yearArray.lastObject]) {
            return self.comp.month;
        }
        return self.monthArray.count;
    } else {
        // 如果是今年今月返回当前已过天数
        if ([self.yearArray[yearIndex] isEqualToString:self.yearArray.lastObject] && self.comp.month - 1 == monthIndex) {
            return self.comp.day;
        }
        switch (monthIndex + 1) {
            case 2:{
                NSString *pickerYear = ((UILabel *)[self.datePicker viewForRow:yearIndex forComponent:0]).text;
                // 需要考虑闰年闰月情况
                if ([self isleapYear:[pickerYear integerValue]]) {
                    return 29;
                } else {
                    return 28;
                }
            }
            case 4:
            case 6:
            case 9:
            case 11:
                return 30;
            default:
                return 31;
        }
    }
}
```
UIPickerView Delegate（部分）
```
// 这里的逻辑主要是用于实时刷新数据源，保证可选范围及显示不会出现或超出当日日期（个人认为这部分还有优化的空间，但暂时没有想到太好的逻辑）
- (void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component {
    if (component == 0) {
        yearIndex = row;
        // 重新加载确保是当前年月时不出现多余可选范围
        [pickerView reloadComponent:1];
        [pickerView reloadComponent:2];
        // 选择年份时月日超出今日日期则自动滚到今天
        if ([self.yearArray[row] isEqualToString:self.yearArray.lastObject]) {
            if (self.comp.month - 1 < monthIndex) {
                monthIndex = self.comp.month - 1;
                [self selectActionToPickerView:pickerView row:monthIndex inComponent:1];

                dayIndex = self.comp.day - 1;
                [self selectActionToPickerView:pickerView row:dayIndex inComponent:2];
            }
            // 这块新手的话可能会觉得有些多余，我就问你：月份没超日期超了你怎么办？！
            if (self.comp.month - 1 == monthIndex && self.comp.day - 1 < dayIndex) {
                dayIndex = self.comp.day - 1;
                [self selectActionToPickerView:pickerView row:dayIndex inComponent:2];
            }
        }
    } else if (component == 1) {
        monthIndex = row;
        // 重新加载确保是当前年月时不出现多余可选范围
        [pickerView reloadComponent:2];
        if (monthIndex + 1 == 4 || monthIndex + 1 == 6 || monthIndex + 1 == 9 || monthIndex + 1 == 11) {
            if (dayIndex + 1 == 31) {
                dayIndex--;
            }
        } else if (monthIndex + 1 == 2) {
            if (dayIndex + 1 > 28) {
                dayIndex = 27;
            }
        }
        // 选择月份时月日超出今日日期则自动滚到今天
        NSString *pickerYear = ((UILabel *)[pickerView viewForRow:yearIndex forComponent:0]).text;
        if ([pickerYear isEqualToString:self.yearArray.lastObject] && self.comp.month - 1 < monthIndex) {
            monthIndex = self.comp.month - 1;
            dayIndex = self.comp.day - 1;
            [self selectActionToPickerView:pickerView row:monthIndex inComponent:1];
            [self selectActionToPickerView:pickerView row:dayIndex inComponent:2];
        }

        [pickerView selectRow:dayIndex inComponent:2 animated:YES];
    } else {
        dayIndex = row;
        // 选择日期时超出今日日期则自动滚到今天
        NSString *pickerYear = ((UILabel *)[pickerView viewForRow:yearIndex forComponent:0]).text;
        NSString *pickerMonth = ((UILabel *)[pickerView viewForRow:monthIndex forComponent:1]).text;
        if ([pickerYear isEqualToString:self.yearArray.lastObject] && [pickerMonth isEqualToString:self.monthArray[self.comp.month - 1]] && self.comp.day - 1 < dayIndex) {
            dayIndex = self.comp.day - 1;
            [self selectActionToPickerView:pickerView row:dayIndex inComponent:2];
        }
        [pickerView selectRow:dayIndex inComponent:2 animated:YES];
    }
}
```

至此，主要的核心代码及实录已经展示完毕，下面来看一下最终效果：

![demo演示](https://raw.githubusercontent.com/LoserForLoser/CurrentDateChooseDatePicker/master/GIF/QQ20180720-123110-HD.gif?raw=true)

怎么样，还可以吧，根本不会有任何多余的日期出现，看都看不到！滑动多余日期玩？不存在的！

本文源码：[CurrentDateChooseDatePicker](https://github.com/LoserForLoser/CurrentDateChooseDatePicker)

欢迎大家提问指教！
