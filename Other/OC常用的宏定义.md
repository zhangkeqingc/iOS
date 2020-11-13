



# OC常用的宏定义



#### 宏定义的优缺点

**优点**

```undefined
提高了程序的可读性，同时也方便进行修改，用户只需要在一处定义，多处使用，修改也只需要修改一处
提高程序的运行效率：使用带参的宏定义既可完成函数调用的功能，又能避免函数的出栈与入栈操作，减少系统开销，提高运行效率，如果有一个函数会在工程中频繁使用，可以考虑一下宏定义
```

**缺点**

```undefined
由于是直接嵌入的，所以代码可能相对多一点
嵌套定义过多可能会影响程序的可读性，而且很容易出错；
对带参的宏而言，由于是直接替换，并不会检查参数是否合法，存在安全隐患。
预编译语句仅仅是简单的值代替，缺乏类型的检测机制
```

**示例**

```
#define APP_VERSION  [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleShortVersionString"]

// System Version
#define SYSTEM_VERSION                             ([[[UIDevice currentDevice] systemVersion] floatValue])
#define SYSTEM_VERSION_EQUAL_TO(v)                 ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedSame)
#define SYSTEM_VERSION_HIGHER_THAN(v)              ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedDescending)
#define SYSTEM_VERSION_EQUAL_TO_OR_HIGHER_THAN(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
#define SYSTEM_VERSION_LOWER_THAN(v)               ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)
#define SYSTEM_VERSION_EQUAL_TO_OR_LOWER_THAN(v)   ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedDescending)

// Color
#define RGB(r,g,b)           [UIColor colorWithRed:r / 255.f green:g / 255.f blue:b / 255.f alpha:1.f]
#define RGBA(r,g,b,a)        [UIColor colorWithRed:r / 255.f green:g / 255.f blue:b / 255.f alpha:a]
#define RGB_HEX(hex)         RGBA((float)((hex & 0xFF0000) >> 16),(float)((hex & 0xFF00) >> 8),(float)(hex & 0xFF),1.f)
#define RGBA_HEX(hex,a)      RGBA((float)((hex & 0xFF0000) >> 16),(float)((hex & 0xFF00) >> 8),(float)(hex & 0xFF),a)
#define COLOR_LIGHT_BLUE     RGB_HEX(0x7f8b97)
#define COLOR_DEEP_BLUE      RGB_HEX(0x00b3d6)

// Language
#define CURRENT_LANGUAGE     ([[NSLocale preferredLanguages] objectAtIndex:0])
#define IS_LANGUAGE(l)       [CURRENT_LANGUAGE hasPrefix:l]
#define IS_LANGUAGE_EN       IS_LANGUAGE(@"en")

// Font
#define FONT_SOFIA_MEDIUM(s) [UIFont fontWithName:@"SofiaProSoft-Medium" size:s]
#define FONT_SOFIA_SOFT(s)   [UIFont fontWithName:@"SofiaProSoft" size:s]

// Screen
#define SCREEN_SIZE          [[UIScreen mainScreen] bounds].size
#define SCREEN_WIDTH         [[UIScreen mainScreen] bounds].size.width
#define SCREEN_HEIGHT        [[UIScreen mainScreen] bounds].size.height

// GCD
#define GCD_GLOBAL(block)   dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), block)
#define GCD_MAIN(block)     dispatch_async(dispatch_get_main_queue(), block)
//如果你设置成Yes，那么你的waring就等于error，编译不了的哦。
```





```
// 真机
#if TARGET_OS_IPHONE
#endif
// 模拟器
#if TARGET_IPHONE_SIMULATOR
#endif

#define IOS8_OR_LATER ([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0)

#define RGBCOLOR(r,g,b) [UIColor colorWithRed:(r)/255.0f green:(g)/255.0f blue:(b)/255.0f alpha:1]
#define RGBACOLOR(r,g,b,a) [UIColor colorWithRed:(r)/255.0f green:(g)/255.0f blue:(b)/255.0f alpha:(a)]
#define HEXCOLOR(rgbValue) [UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0]

#define SCREEN_HEIGHT [[UIScreen mainScreen] bounds].size.height
#define SCREEN_WIDTH  [[UIScreen mainScreen] bounds].size.width

#ifdef DEBUG
#define JSLog(FORMAT, ...) fprintf(stderr,"%s:%d\t%s\n",[[[NSString stringWithUTF8String:__FILE__] lastPathComponent] UTF8String], __LINE__, [[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);
#else
#define JSLog(FORMAT, ...) nil
#endif


#define JSDefault [NSUserDefaults standardUserDefaults]


//是否为空或是[NSNull null]
#define NotNilAndNull(_ref)  (((_ref) != nil) && (![(_ref) isEqual:[NSNull null]]))
#define IsNilOrNull(_ref)   (((_ref) == nil) || ([(_ref) isEqual:[NSNull null]]))
//字符串是否为空
#define IsStringEmpty(_ref)    (((_ref) == nil) || ([(_ref) isEqual:[NSNull null]]) ||([(_ref)isEqualToString:@""]))
//数组是否为空
#define IsArrayEmpty(_ref)    (((_ref) == nil) || ([(_ref) isEqual:[NSNull null]]) ||([(_ref) count] == 0))


#define Foreground_Begin  dispatch_async(dispatch_get_main_queue(), ^{
#define Foreground_End    });

#define Background_Begin  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{\
                                @autoreleasepool {
#define Background_End          }\
                          });


#if __has_feature(objc_arc)
// ARC
#else
// 非ARC
#endif

#define degreesToRadian(x) (M_PI * (x) / 180.0)
#define radianToDegrees(radian) (radian*180.0)/(M_PI)


#undef  AS_SINGLETON
#define AS_SINGLETON( __class ) \
+ (__class *)sharedInstance;

#undef  DEF_SINGLETON
#define DEF_SINGLETON( __class ) \
+ (__class *)sharedInstance \
{ \
static dispatch_once_t once; \
static __class * __singleton__; \
dispatch_once( &once, ^{ __singleton__ = [[__class alloc] init]; } ); \
return __singleton__; \
}

#define WeakSelf(type)  __weak typeof(type) weak##type = type;
#define StrongSelf(type)  __strong typeof(type) type = weak##type;


#define ViewBorderRadius(View, Radius, Width, Color)\
\
[View.layer setCornerRadius:(Radius)];\
[View.layer setMasksToBounds:YES];\
[View.layer setBorderWidth:(Width)];\
[View.layer setBorderColor:[Color CGColor]]

#define DegreesToRadian(x) (M_PI * (x) / 180.0)
#define RadianToDegrees(radian) (radian*180.0)/(M_PI)


//获取temp
#define kPathTemp NSTemporaryDirectory()
//获取沙盒 Document
#define kPathDocument [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
//获取沙盒 Cache
#define kPathCache [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject]

```



