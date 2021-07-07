# MGJRouter
一个高效/灵活的 iOS URL Router

### 2021-02-02 更新
使用中发现 在 + (void)openURL:(NSString *)URL withUserInfo:(NSDictionary *)userInfo completion:(void (^)(id result))completion
中传入 nil 会出闪退的问题

### 2015-08-22 更新

#### 添加了同步获取 Object 的方法

有些场景我们可能需要根据 URL 获取某个 Object，所以就添加了这个方法

```objc
UIView *searchTopBar = [MGJRouter objectForURL:@"search_top_bar"];
```

## 为什么要再造一个轮子？
已经有几款不错的 Router 了，如 [JLRoutes](https://github.com/joeldev/JLRoutes), [HHRouter](https://github.com/Huohua/HHRouter), 但细看了下之后发现，还是不太满足需求。

JLRoutes 的问题主要在于查找 URL 的实现不够高效，通过遍历而不是匹配。还有就是功能偏多。

HHRouter 的 URL 查找是基于匹配，所以会更高效，MGJRouter 也是采用的这种方法，但它跟 ViewController 绑定地过于紧密，一定程度上降低了灵活性。

于是就有了 MGJRouter。

## 安装

```
pod 'MGJRouter', '~>0.9.0'
```

## 使用姿势

### 最基本的使用

```objc
[MGJRouter registerURLPattern:@"mgj://foo/bar" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameterUserInfo:%@", routerParameters[MGJRouterParameterUserInfo]);
}];

[MGJRouter openURL:@"mgj://foo/bar"];
```

当匹配到 URL 后，`routerParameters` 会自带几个 key

```objc
extern NSString *const MGJRouterParameterURL;
extern NSString *const MGJRouterParameterCompletion;
extern NSString *const MGJRouterParameterUserInfo;
```

### 处理中文也没有问题

```objc
[MGJRouter registerURLPattern:@"mgj://category/家居" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameters:%@", routerParameters);
}];

[MGJRouter openURL:@"mgj://category/家居"];
```

### Open 时，可以传一些 userinfo 过去

```objc
[MGJRouter registerURLPattern:@"mgj://category/travel" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameters[MGJRouterParameterUserInfo]:%@", routerParameters[MGJRouterParameterUserInfo]);
    // @{@"user_id": @1900}
}];

[MGJRouter openURL:@"mgj://category/travel" withUserInfo:@{@"user_id": @1900} completion:nil];
```

### 如果有可变参数（包括 URL Query Parameter）会被自动解析

```objc
[MGJRouter registerURLPattern:@"mgj://search/:query" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameters[query]:%@", routerParameters[@"query"]); // bicycle
    NSLog(@"routerParameters[color]:%@", routerParameters[@"color"]); // red
}];

[MGJRouter openURL:@"mgj://search/bicycle?color=red"];
```

### 定义一个全局的 URL Pattern 作为 Fallback

```objc
[MGJRouter registerURLPattern:@"mgj://" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"没有人处理该 URL，就只能 fallback 到这里了");
}];

[MGJRouter openURL:@"mgj://search/travel/china?has_travelled=0"];
```

### 当 Open 结束时，执行 Completion Block

```objc
[MGJRouter registerURLPattern:@"mgj://detail" toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"匹配到了 url, 一会会执行 Completion Block");

    // 模拟 push 一个 VC
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.25 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        void (^completion)() = routerParameters[MGJRouterParameterCompletion];
        if (completion) {
            completion();
        }
    });
}];

[MGJRouter openURL:@"mgj://detail" withUserInfo:nil completion:^{
    [self appendLog:@"Open 结束，我是 Completion Block"];
}];
```

### 生成 URL

URL 的处理一不小心，就容易散落在项目的各个角落，不容易管理。比如注册时的 pattern 是 `mgj://beauty/:id`，然后 open 时就是 `mgj://beauty/123`，这样到时候 url 有改动，处理起来就会很麻烦，不好统一管理。

所以 MGJRouter 提供了一个类方法来处理这个问题。

```objc
+ (NSString *)generateURLWithPattern:(NSString *)pattern parameters:(NSArray *)parameters;
```

使用方式

```objc
#define TEMPLATE_URL @"mgj://search/:keyword"

[MGJRouter registerURLPattern:TEMPLATE_URL  toHandler:^(NSDictionary *routerParameters) {
    NSLog(@"routerParameters[keyword]:%@", routerParameters[@"keyword"]); // Hangzhou
}];

[MGJRouter openURL:[MGJRouter generateURLWithPattern:TEMPLATE_URL parameters:@[@"Hangzhou"]]];
}
```

这样就可以在一个地方定义所有的 URL Pattern，使用时，用这个方法生成 URL 就行了。


## 协议

MGJRouter 被许可在 MIT 协议下使用。查阅 LICENSE 文件来获得更多信息。
