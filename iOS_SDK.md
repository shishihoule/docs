# Yunba iOS SDK 使用文档

## 导入 SDK 
下载的 lib文件夹加入到项目工程中，在需要使用YunBa的代码中引入YunBaService.h，添加依赖库SystemConfiguration。

## 添加使用代码
初始化 YunBa SDK。请在您的代码入口函数didFinishLaunchingWithOptions处添加如下代码：

    [YunBaService setupWithAppkey:AppKey LogLevel:kYBLogLevelDebug];


## 添加 Message Received 代码

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onMessageReceived:) name:kYBDidReceiveMessageNotificationKey object:nil];


###  自定义消息处理函数处理 Publish 消息代码示例  

    - (void) onMessageReceived:(NSNotification *)notification {
        YBMessage *message = [notification object];
        NSString *payloadString = [[NSString alloc] initWithData:[message data] encoding:NSUTF8StringEncoding];
        NSLog(@"received string: %@ on topic %@", payloadString, [message topic]);
    }


## API - subscribe

### 功能
App 可以增加订阅一个Topic, 以便可以接收来自 Topic 的 Message。

### 函数原型

     + (void)subscribeToTopic:(NSString *)topic atLevel:(UInt8)qosLevel resultBlock:(YBResultBlock)resultBlock;

### 参数说明
* topic: app 订阅的的主题，topic 只支持英文数字下划线，长度不超过50个字符。
* qosLevel: 服务质量等级，具体参考服务质量等级说明。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [YunBaService subscribeToTopic:topic atLevel:qosLevel resultBlock:^(BOOL succ, NSError *error){
        if (succ) {
            NSLog(@"subscribe to topic succ: %@", topic);
        } else {
            NSLog(@"subscribe to topic failed due to : %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];

## API - unsubscribe

### 功能
App 可以取消订阅一个 Topic, 以便取消接收来自 Topic 的 Message.

### 函数原型


     + (void)unsubscribeTopic:(NSString*)topic resultBlock:(MbResultBlock)resultBlock;


### 参数说明
* topic: app 订阅的的主题，topic 只支持英文数字下划线，长度不超过50个字符。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [YunBaService unsubscribeTopic:topic resultBlock:^(BOOL succ, NSError *error){
        if (succ) {
            NSLog(@"unsubscribe to topic succ: %@", topic);
        } else {
            NSLog(@"unsubscribe to topic failed due to: %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];



## API - Publish

### 功能
App 可以向 Topic 发送消息, 那么任何订阅此 Topic 的 Client 都会接受到消息。

### 函数原型

     + (void)publishData:(NSData*)data ToTopic:(NSString*)topic atLevel:(UInt8)qosLevel retain:(BOOL)retainFlag resultBlock:(MbResultBlock)resultBlock;

### 参数说明
* topic: app 待发布消息的主题，只支持英文数字下划线，长度不超过50个字符。
* message: 向对应 topic 的订阅者发布的消息。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example


    [YunBaService publishData:data ToTopic:topic atLevel:qosLevel retain:isRetained
                  resultBlock:^(BOOL succ, NSError *error){
        if (succ) {
            NSLog(@"publish data: %@ to topic: %@ succ", data, topic);
        } else {
            NSLog(@"publish to topic failed due to: %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];


## API - Store DeviceToken

### 功能
App 将DeviceToken 存储在YunBa的云端， 那么可以通过YunBa发送APNs通知。

### 函数原型

     + (void)storeDeviceToken:(NSData *)token resultBlock:(YBResultBlock)resultBlock;

### 参数说明
* token: 通过注册APNs而获取到的对应设备的device token。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];     //注册APNs，申请获取device token

    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        NSLog(@"get Device Token: %@", [NSString stringWithFormat:@"Device Token: %@", deviceToken]);
        [YunBaService storeDeviceToken:deviceToken resultBlock:^(BOOL succ, NSError *error) {
            if (succ) {
                NSLog(@"store device token to YunBa succ");
            } else {
                NSLog(@"store device token to YunBa failed due to : %@", error);
            }
        }];
    }


## API - Report

### 功能
App  可以调用此函数来上报客户端的行为，如打开通知栏次数，按钮点击次数，资源下载成功等等行为。

### 函数原型

     + (void)report:(NSString *)action withData:(NSData *)data;

### 参数说明
* action: app 需要统计的行为，如打开通知栏，下载资源成功等等。
* data: 想对应 action 的附加数据，以满足统计相关的其他业务需求。

Code Example

    [YunBaService report:action withData:data];



## API - SetAlias

### 功能
App 可以调用此函数来绑定账号，用户名，每个用户只能指定一个别名。

### 函数原型

     + (void)setAlias:(NSString *)alias resultBlock:(YBResultBlock)resultBlock;

### 参数说明
* alias: 用户设置的别名信息，只支持英文数字下划线，长度不超过50个字符.
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [YunBaService setAlias:alias resultBlock:^(BOOL succ, NSError *error) {
        if (succ) {
            NSLog(@"set alias succ");
        } else {
            NSLog(@"set alias failed due to : %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];



## API - GetAlias

### 功能
App 可以调用此函数来获取当前用户的别名。

### 函数原型

     + (void)getAlias:(YBStringResultBlock)stringResultBlock;

### 参数说明
* stringResultBlock: API 回调接口， 可通过返回的 NSError *error获取错误代码(kYBErrorNoError表示成功)及原因。

Code Example

    [YunBaService getAlias:^(NSString *res, NSError *error) {
        if (error.code == kYBErrorNoError) {
            NSLog(@"get alias succ:%@", res);
        } else {
            NSLog(@"get alias failed due to : %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];



## 添加 Presence Received 代码

        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(onPresence:) name:kYBDidReceivePresenceNotificationKey object:nil];


###  自定义消息处理函数处理 onPresence 消息代码示例  

    - (void)onPresence:(NSNotification *)notification {    
        YBPresenceEvent *presence = [notification object];
        NSString *theTime = [NSDateFormatter localizedStringFromDate:[NSDate dateWithTimeIntervalSince1970:[presence time]/1000] dateStyle:NSDateFormatterMediumStyle timeStyle:NSDateFormatterMediumStyle];
        NSLog(@"new presence, action=%@, topic=%@, alias=%@, time=%@", [presence action], [presence topic], [presence alias], theTime);
    }


## API - Subscribe Presence

### 功能
App 可以订阅某个频道上的用户的上、下线及(取消)订阅事件通知。

### 函数原型

     + (void)subscribePresenceToTopic:(NSString *)topic resultBlock:(YBResultBlock)resultBlock;

### 参数说明
* topic: app 订阅的的目标用户所在频道主题，topic 只支持英文数字下划线，长度不超过50个字符。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [YunBaService subscribePresenceToTopic:topic resultBlock:^(BOOL succ, NSError *error) {
        if (succ) {
            NSLog(@"subscribe presence to topic:%@ succ", topic);
        } else {
            NSLog(@"subscribe presence failed due to : %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];



## API - Unsubscribe Presence

### 功能
App 可以取消订阅某个频道上的用户的上、下线及(取消)订阅事件通知。

### 函数原型

     + (void)unsubscribePresenceToTopic:(NSString *)topic resultBlock:(YBResultBlock)resultBlock;

### 参数说明
* topic: app 订阅的的目标用户所在频道主题，topic 只支持英文数字下划线，长度不超过50个字符。
* resultBlock: API 回调接口， 可通过返回的BOOL succ判断结果的成功与否, NSError *error获取错误原因。

Code Example

    [YunBaService unsubscribePresenceToTopic:topic resultBlock:^(BOOL succ, NSError *error) {
        if (succ) {
            NSLog(@"unsubscribe presence to topic:%@ succ", topic);
        } else {
            NSLog(@"unsubscribe presence failed due to : %@, recovery suggestion: %@", error, [error localizedRecoverySuggestion]);
        }
    }];