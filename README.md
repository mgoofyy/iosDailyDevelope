#####导航栏背景透明 
```
[self.navigationController.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];
```
#####导航栏底部线清除
```
self.navigationController.navigationBar.barStyle = UIBarStyleBlack; 
self.navigationController.navigationBar.translucent = YES; 
[self.navigationController.navigationBar setShadowImage:[UIImage new]];
```
#####导航栏底部线清除
```
UIView *statusBarView = [[UIView alloc] initWithFrame:CGRectMake(0, -20, ScreenWidth, 20)];
statusBarView.backgroundColor = HRMainColorGreen;
[self.navigationBar addSubview:statusBarView];
```
#####生成指定大小的纯色Image  eg:5*5px
```
+ (UIImage *)imageFromColor:(UIColor *)color {
    
    CGRect rect = CGRectMake(0, 0, 5, 5);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context,rect);
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext(); return img;
}
```
#####关于RAC的一点摘录笔记
```
在信号达到订阅者之前执行block里面的操作,用于动态注入signal的操作(什么side effect...你叫我怎么翻译,google翻译机吖,副作用,注入操作?这个好,但好像不太对路...又是各种渣翻译)
- (RACSignal *)doNext:(void (^)(id x))block;

这个看上面的就知道,在error到达订阅者操作前执行block里面的注入操作
- (RACSignal *)doError:(void (^)(NSError *error))block;

同上,大神手痒,跟我懒无关
- (RACSignal *)doCompleted:(void (^)(void))block;

如果interval设置时间为3秒,这样我们在3秒过后获取到在这3秒中最后一个信号值,然后执行订阅者的block,每隔3秒执行一次一个定时的东西吧
- (RACSignal *)throttle:(NSTimeInterval)interval;

比上面那个更加复杂点,在收到信号值时在后面的block返回一个布尔值来表示该值是否通过,然后还是根据最后一个值,在间隔时间过去后给订阅者执行操作
- (RACSignal *)throttle:(NSTimeInterval)interval valuesPassingTest:(BOOL (^)(id next))predicate;

延迟接收信号值,时间长短由interval决定,没其他太特别的地方
- (RACSignal *)delay:(NSTimeInterval)interval;

信号发出completed的值后重复发送之前的信号值
- (RACSignal *)repeat;

有点try-catch-final的感觉,使用结果是initially以后先执行最后的一个signal的initially的block,然后依次执行之前写的block,当信号发出completed则执行final的block
- (RACSignal *)initially:(void (^)(void))block;
- (RACSignal *)finally:(void (^)(void))block;

在未来的一段时间内缓冲信号值,然后到时间后发放,如果多个值则用RACTuple
- (RACSignal *)bufferWithTime:(NSTimeInterval)interval onScheduler:(RACScheduler*)scheduler;

收集信号值,直到信号发出completed,然后用一个NSArray返回给订阅者
- (RACSignal *)collect;

将completed前的最后N个返回给订阅者
- (RACSignal *)takeLast:(NSUInteger)count;

将两个信号合并,当其中一个有新的信号值的时候,如另外一个已经有信号则,则取最后收到的一个,打包成RACTuple一并发给订阅者,如冇则不触发订阅者的操作
- (RACSignal *)combineLatestWith:(RACSignal *)signal;

类方法,跟上面的方法相似,但是可以多个信号合并在一起
+ (RACSignal *)combineLatest:(id)signals;

多个信号合并成一个信号,发一次信号值,触发一次订阅者操作
+ (RACSignal *)merge:(id)signals;

这个可以砍砍RACStream的- (instancetype)flatten;方法,其实跟merge差不多,但是别人是类方法,这个是一个实例方法
- (RACSignal *)flatten:(NSUInteger)maxConcurrent;

然后,然后干嘛呢,就是当那些信号发送completed后再接收一个信号,信号在then的block中返回,但是动态的信号貌似就不行了,不知道为什么
 
- (RACSignal *)then:(RACSignal * (^)(void))block;

这货的注释真的有点变态'Concats the inner signals of a signal of signals.'链接一个信号中的信号,我要是不上google我能懂么,拜托!这东西是说如果一个信号里面包含多个信号,而这些信号又没有联系的同时就可以使用concat来串联起来,所以一开始我用merge来生成信号然后用concat就狂报错...
 
- (RACSignal *)concat;

注释很少,也确实跟之前看过的方法有些相似,就是从某个信号值(别人叫玻璃球,这回让人误会的好不好,所以我还是这么渣翻译)开始,然后跟ruduce的方法差不多,就是让你决定某次的值该返回哪一个,running是上次那个囖
 
- (RACSignal *)aggregateWithStart:(id)start reduce:(id (^)(id running, id next))reduceBlock;

这个跟上面那个差不多,就是在最前面的一个信号值前先自己在factory里返回一个出来,然后再在后面的block决定返回哪个信号值
 
- (RACSignal *)aggregateWithStartFactory:(id (^)(void))startFactory reduce:(id (^)(id running, id next))reduceBlock;

自动将下一个值设置给对应的路径,就是各种介绍里面的绑定动态值的方法,只不过别人用的是宏
 
- (RACDisposable *)setKeyPath:(NSString *)keyPath onObject:(NSObject *)object;
 
- (RACDisposable *)setKeyPath:(NSString *)keyPath onObject:(NSObject *)object nilValue:(id)nilValue;

时间间隔的信号,Leeway是偏差,余地,缓冲的意思,但是试过几个数值都不太明显
 
+ (RACSignal *)interval:(NSTimeInterval)interval onScheduler:(RACScheduler *)scheduler;
 
+ (RACSignal *)interval:(NSTimeInterval)interval onScheduler:(RACScheduler *)scheduler withLeeway:(NSTimeInterval)leeway;

 
- (RACSignal *)takeUntil:(RACSignal *)signalTrigger;

try-catch的RAC方法
 
- (RACSignal *)try:(BOOL (^)(id value, NSError **errorPtr))tryBlock;
 
- (RACSignal *)tryMap:(id (^)(id value, NSError **errorPtr))mapBlock;
- (RACSignal *)catch:(RACSignal * (^)(NSError *error))catchBlock;
 
- (RACSignal *)catchTo:(RACSignal *)signal;

///返回信号中第一个next里面的值
- (id)first;

///返回信号中的第一个,如果第一个就是complete或者error就返回defaultValue里面的值
- (id)firstOrDefault:(id)defaultValue;

///跟上面两一样,后面success是用来控制返回结果的
- (id)firstOrDefault:(id)defaultValue success:(BOOL *)success error:(NSError **)error;

///等待信号完成,如果返回NO,则error被设置
- (BOOL)waitUntilCompleted:(NSError **)error;

///类方法,返回一个延迟的信号.直到被订阅.这可以被用作把一个热信号转换成冷信号(...我都不知道怎么说了,信号本身就是不订阅就没其他操作,一旦订阅就开始进行block里面的操作,这用跟不用这个方法都一样吖)
 
+ (RACSignal *)defer:(RACSignal * (^)(void))block;

///接受者必须是多个信号组成的信号,信号中所有的信息,包括complete和error都会发送到最后一个信号中
 
- (RACSignal *)switchToLatest;

///跟上面的方法差不多,cases是多个signal的组合,default不用说,最后返回一个组装好的signal
 
+ (RACSignal *)switch:(RACSignal *)signal cases:(NSDictionary *)cases default:(RACSignal *)defaultSignal;

///每次发送信号都由boolSignal来决定,YES->tureSignal,NO->falseSignal,最后组装成signal
 
+ (RACSignal *)if:(RACSignal *)boolSignal then:(RACSignal *)trueSignal else:(RACSignal *)falseSignal;

///将值组装成一个NSArray
 
- (NSArray *)toArray;

///返回一个队列跟RACSequence里面的singal相反
 
@property (nonatomic, strong, readonly) RACSequence *sequence;

///RACMulticastConnection和RACReplaySubject的一些方法,研究了很久都不太懂,相关的东西也google过还是没明白,看来还是得用渣渣的英语来问问大神们
 
- (RACMulticastConnection *)publish;
 
- (RACMulticastConnection *)multicast:(RACSubject *)subject;
 
- (RACSignal *)replay;
 
- (RACSignal *)replayLast;
 
- (RACSignal *)replayLazily;

///设置等待超时
 
- (RACSignal *)timeout:(NSTimeInterval)interval onScheduler:(RACScheduler *)scheduler;

///在设置的调度中发送信号值,但操作封包依然在原来的调度里进行
 
- (RACSignal *)deliverOn:(RACScheduler *)scheduler;

///在设置的调度中发送信号和执行封包里的操作.
 
- (RACSignal *)subscribeOn:(RACScheduler *)scheduler;

///将信号的值打包成一个RACGroupedSignal,transformBlock用来map相关的值
 
- (RACSignal *)groupBy:(id<NSCopying> (^)(id object))keyBlock transform:(id (^)(id object))transformBlock;
 
- (RACSignal *)groupBy:(id<NSCopying> (^)(id object))keyBlock;

///当信号中有任意的值出现,就直接用YES返回
 
- (RACSignal *)any;

///在predicateBlock里面浏览值来确定是否返回YES
 
- (RACSignal *)any:(BOOL (^)(id object))predicateBlock;

///如果predicateBlock里全部都返回yes,则YES返回到信号里
 
- (RACSignal *)all:(BOOL (^)(id object))predicateBlock;

///当error发生时重新订阅信号,retryCount有次数限制
 
- (RACSignal *)retry:(NSInteger)retryCount;
 
- (RACSignal *)retry;

///注释上是说当sampler发送一个值后从接受者中发送最后一个值.如果sampler触发得比接受者还频繁,则返回的信号会重复发送值,其实我也没试出来...
- (RACSignal *)sample:(RACSignal *)sampler;

///忽略所有的值,除了complete和error
- (RACSignal *)ignoreValues;

///返回一个将所有的值都转换成RACEvent对象的信号
- (RACSignal *)materialize;

///上面的相反方法,将RACEvent对象转转成原来的值
- (RACSignal *)dematerialize;

///not是对于信号值是BOOL类型的相反值,而and和or就对RACTruple里面的值进行逻辑运算,然后返回
- (RACSignal *)not;
- (RACSignal *)and;
 
- (RACSignal *)or;
```
    


