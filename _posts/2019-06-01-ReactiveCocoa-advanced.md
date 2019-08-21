---
layout:     post
title:      ReactiveCocoa advanced
subtitle:   Functional programming framework ReactiveCocoa advanced
date:       2019-06-01
author:     adayxiang
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - ReactiveCocoa
---
# Foreword

 

> In [ the article ] (http://qiubaiying.github.io/2016/12/26/ReactiveCocoa- base /) introduced in ** ReactiveCocoa ** basics , then we come in-depth introduction ** ReactiveCocoa ** and its use in **MVVM** .

 

 

![ReactiveCocoa Advanced Mind Map ](https://ww3.sinaimg.cn/large/006y8lVagw1fbgye3re5xj30je0iomz8.jpg)

# Common method

 

 

#### Operation

 

All signals ( RACSignal ) can be manipulated because all methods of operation are defined in RACStream.h , so there is an operational handling method as long as the RACStream is inherited .

#### Operational thinking

 

Using the Hook idea, Hook is a technique for changing the execution results of an API ( application programming interface: method ) .

 

Hook Usage: The technique of intercepting API calls.

 

About Hook knowledge I can see this blog [ " in Objective-C Runtime some basic use" ] (http://www.jianshu.com/p/ff114e69cc0a) in * Replace the implementation code * a ,

 

Hook principle: Execute your own method and change the output of the result before calling each API to return the result.

 

#### 操作方法

 

#### **bind** (Binding) - Core Method of ReactiveCocoa

 

** The core method of ReactiveCocoa** operation is **bind** (binding) , and it is also the core development method in RAC . The previous development method is assignment, but with RAC development, you should focus on the binding, that is, when you create an object, you can bind what you want to do later, instead of doing things after the assignment. .

 

For example, to display the data to the control, the previous method is to rewrite the control's `setModel` method. With RAC, you can bind the data when you create the control at the beginning.

 

- ** role **

 

              The bottom layer of RAC is called **bind** . In development, **bind** method is rarely used directly . **bind** belongs to the underlying method in RAC . We only need to call the encapsulated method, **bind* * Used to understand .

 

- **bind method use step **

     1. Pass in a block with a return value of `RACStreamBindBlock` .

     2. description of a `RACStreamBindBlock` type ` bindBlock` as block return value.

     3. Describe a signal that returns a result as the return value of `bindBlock` .

 

     Note: The processing of signal results is done in bindblock .

- **bind method parameter **             

             

              **RACStreamBindBlock**:

`typedef RACStream * (^RACStreamBindBlock)(id value, BOOL *stop);`

 

     ` Parameter one (value): indicates the original value of the received signal, has not been processed yet.

 

     ` Parameter 2 (*stop)`: Used to control the binding block . If *stop = yes, the binding will end.

 

     ` Return value ` : signal processing to do, this signal is returned out by generally used `RACReturnSignal`, need to manually import header file ` RACReturnSignal.h`

 

- ** Use **

 

              Suppose you want to listen to the contents of the text box, and each time you output the result, splicing a piece of text in the text box " output: "

 

              - Use a packaged method: splicing after returning the result.

 

                            ```

                            [_textField.rac_textSignal subscribeNext:^(id x) {

                           

                                          // After returning the result, splicing the output:

                                          NSLog (@" output : %@", x);

                           

                            }];

                            ```

 

 

              - Method 2 : Use the `bind` method in RAC to do the processing, and splicing before returning the result.

               

                            Here you need to manually import `#import <ReactiveCocoa/RACReturnSignal.h>` to use `RACReturnSignal`

 

                            ```             

                            [[_textField.rac_textSignal bind:^RACStreamBindBlock{

                               // When to call :

                               // block role : indicates that a signal is bound .

                           

                               Return ^RACStream *(id value, BOOL *stop){

                           

                                   / / When to call block: When the signal has a new value issued, it will come to this block .

                           

                                   // block role : do the return value processing

                           

                                   // Do a good job of splicing the output before returning the result :

                                   Return [RACReturnSignal return:[NSString stringWithFormat:@" output :%@",value]];

                               };

                           

                            }] subscribeNext:^(id x) {

                           

                               NSLog(@"%@",x);

                           

                            }];

 

                            ```

 

- ** Bottom implementation **

     1. The source signal calls bind and a new binding signal is created.

     2. When the binding signal is subscribed, it will call `didSubscribe` in the binding signal to generate a `bindingBlock` .

     3. When the source signal has content, it will be passed to the `bindingBlock` handler, calling `bindingBlock(value,stop)`

     4. Call `bindingBlock(value, stop)` , which will return a signal `RACReturnSignal` for the completion of the content processing .

     5. Subscribe to `RACReturnSignal` and you will get the subscriber of the binding signal and send the processed signal content.

 

     Note : Different subscribers, save different nextBlock , when looking at the source code, be sure to see which subscriber is.

 

#### MAP

 

The mapping is mainly implemented by these two methods: **flattenMap**, **Map**, which maps the source signal content to new content.

 

###### flattenMap

 

- ** role **

 

              Map the content of the source signal to a new signal, which can be of any type

 

- ** Usage steps **

 

     1. Pass in a block , block type is the return value `RACStream` , parameter value

     2. The parameter value is the content of the source signal, and the content of the source signal is processed.

     3. Wrap it into a `RACReturnSignal` signal and return it.

 

 

 

- ** Use **

 

              Listen to the contents of the text box and remap the structure to a new value .

             

              ```

              [[_textField.rac_textSignal flattenMap:^RACStream *(id value) {

 

        // block call timing: when the signal source is sent

 

        // block role: change the content of the signal

 

        / / Return RACReturnSignal

        Return [RACReturnSignal return:[NSString stringWithFormat:@" Signal content: %@", value]];

 

    }] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

    ```

- ** Bottom implementation **

 

     0. **flattenMap** internally calls the `bind` method , the return value of the block in **flattenMap** will be used as the return value of binddBlock in bind .

     1. When the subscription signal is subscribed, `bindBlock` is generated .

     2. When the source signal sends the content, it will call ` bindBlock(value, *stop)`

     3. call `bindBlock` , internal calls will ** flattenMap ** of Bloc K , ** flattenMap ** the block effect: that is packaged into the processed data signal.

     4. The returned signal will eventually be used as the return signal in `bindBlock` as the return signal for `bindBlock` .

     5. Subscribe to the return signal of `bindBlock` , and you will get the subscriber of the binding signal and send the processed signal content.

             

###### Map

 

- ** role **

 

              Map the value of the source signal to a new value

 

             

- ** Usage steps **

     1. Pass in a block, the type is the return object, the parameter is `value`

     2. `value` is the content of the source signal, directly get the content of the source signal for processing

     3. Return the processed content directly, just do not wrap it into a signal, and the returned value is the mapped value.

 

- ** Use **

 

              Listen to the contents of the text box and remap the structure to a new value .

 

    ```

              [[_textField.rac_textSignal map:^id(id value) {

 

       / / After the stitching, return the object

        Return [NSString stringWithFormat:@" Signal content : %@", value];

 

    }] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

              ```

- ** Bottom implementation **:

     0. Map the underlying fact is to call `flatternMa`p,` Map` the block value will be returned as `flatternMap` in block values

     1. When the subscription signal is subscribed, it will generate `bindBlock`

     3. When the source signal sends the content, it will call `bindBlock(value, *stop)`

     4. calls `bindBlock` , internal calls the ` flattenMap of block`

     5. `flattenMap the block` internal calls ` Map` in the block , the `Map` the block returned content packaged return signal

     The signal returned will eventually as `bindBlock` return signal, as ` bindBlock` return signal

     6. Subscribe to the return signal of `bindBlock` , and you will get the subscriber of the binding signal and send the processed signal content.

 

###### The difference between FlatternMap and Map

- ** FlatternMap ** in Block ** return signal ** .

2. ** Map ** in Block ** return the object ** .

3. In development, if the value ** issued by the signal is not the signal ** , the mapping generally uses `Map`

4. If the value ** issued by the signal is signal ** , the mapping generally uses `FlatternMap` .

 

 

 

- `signalOfsignals` with **FlatternMap**

 

              ```

    / / Create a signal in the signal

    RACSubject *signalOfsignals = [RACSubject subject];

    RACSubject *signal = [RACSubject subject];

 

    [[signalOfsignals flattenMap:^RACStream *(id value) {

 

     / / When the signals of signalOfsignals signal will be called

 

        Return value;

 

    }] subscribeNext:^(id x) {

 

        // Only signalOfsignals the signal emitted signal will be called, because the internal subscribed bindBlock signal is returned, which is flattenMap return signal.

        // That is , the signal sent by the flattenMap will be called.

 

        NSLog(@"signalOfsignals : %@",x);

    }];

 

    // Signal signal transmission signal

    [signalOfsignals sendNext:signal];

 

    // Signal transmission content

    [signal sendNext:@"hi"];

             

              ```

             

#### combination

 

The combination is to splicing multiple signals according to certain rules to synthesize new signals.

 

###### concat

 

- ** role **

 

              The ** signal is spliced in ** order , and when multiple signals are sent, there is a sequential received signal.

- ** Bottom implementation **

     1. When the spliced ​​signal is subscribed, the iditched signal's didSubscribe is called.

     2. didSubscribe , will first subscribe to the first source signal ( signalA )

     3. will perform the first source signal ( signalA ) of the didSubscribe

     4. The first source signal ( signalA ) sends a value in didSubscribe , which will call the first source signal ( signalA ) subscriber's nextBlock, which will be sent out by the subscriber of the spliced ​​signal .

     5. The first signal source ( signalA ) didSubscribe transmitting completed, is called a first signal source ( signalA ) subscriber completedBlock, subscribe to a second signal source ( signalB ) which when it is activated ( signalB ).

     6. Subscribing to the second source signal ( signalB ) , performing the second source signal ( signalB ) of the didSubscribe

     7. The second signal source ( signalA ) didSubscribe transmitted values , will be the value sent by the subscriber splicing out signal .

- ** Usage steps **

 

              1. Use `concat:` stitching signal

              2. Subscribe to the splicing signal, the internal will automatically subscribe to the signal in the splicing order

- ** Use **

 

              Splicing signals `signalA` , `signalB` , `signalC`

             

              ```

              RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"Hello"];

 

        [subscriber sendCompleted];

 

        Return nil;

    }];

 

    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"World"];

 

        [subscriber sendCompleted];

 

        Return nil;

    }];

 

    RACSignal *signalC = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"!"];

 

        [subscriber sendCompleted];

 

        Return nil;

    }];

 

    // splicing AB, the signalA spliced to signalB later, signalA transmission is complete, signalB will be activated.

    RACSignal *concatSignalAB = [signalA concat:signalB];

 

    // AB + C

    RACSignal *concatSignalABC = [concatSignalAB concat:signalC];

 

 

    / / Subscribe to the stitched signal , the internal will subscribe to A-> B-> C in order

    // Note: The first signal must be sent and the second signal will be activated ...

    [concatSignalABC subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

              ```

 

###### then

- ** role **

 

              Used to connect two signals, when the first signal is completed, the signal returned by then will be connected .

- ** Bottom implementation **

             

              1. Filter out the value of the previous signal first

              2. Use concat to connect the signal returned by then

             

- ** Use **

 

              ```

   [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

      [subscriber sendNext:@1];

 

      [subscriber sendCompleted];

 

      Return nil;

 

    }] then:^RACSignal *{

 

                    Return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

          [subscriber sendNext:@2];

 

          Return nil;

      }];

 

    }] subscribeNext:^(id x) {

 

      / / Only receive the value of the second signal, which is the value of the then return signal

      NSLog(@"%@", x);

 

    }];

 

    ///

    Output: 2

              ```

- ** Note **

 

              Note that with `then` , the value of the previous signal will be ignored .

 

###### merge

- ** role **

             

              The combined signal , any signal sent by the signal, can be monitored .

- ** Bottom implementation **

 

     1. When the combined signal is subscribed, it will traverse all the signals and send them.

     2. Each time a signal is sent, this signal will be subscribed

     3. That is, once the merged signal is subscribed, it will subscribe to all the signals inside.

     4. As long as a signal is sent, it will be monitored.

- ** Use **

 

              ```

              RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"A"];

 

        Return nil;

    }];

 

    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"B"];

 

        Return nil;

    }];

 

    / / Combined signal , any signal sent data, can be monitored

    RACSignal *mergeSianl = [signalA merge:signalB];

 

    [mergeSianl subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    // output

              2017-01-03 13:29:08.013 ReactiveCocoa Advanced [3627:718315] A

              2017-01-03 13:29:08.014 ReactiveCocoa advanced [3627:718315] B

 

 

              ```

 

###### zip

 

- ** role **

             

              The two signals are compressed into a signal only when the two signals ** while ** when signaled the content, and the content of the two signals are combined into a tuple, it will trigger a compressed stream next event.

- ** Bottom implementation **

             

              1. Define the compressed signal, the internal will automatically subscribe to signalA , signalB

              2. When signalA or signalB sends a signal, it will judge whether signalA or signalB sends a signal, and the value of the first time of each signal will be packaged into a tuple.

                  

- ** Use **

 

              ```

              RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"A1"];

        [subscriber sendNext:@"A2"];

 

        Return nil;

    }];

 

    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"B1"];

        [subscriber sendNext:@"B2"];

        [subscriber sendNext:@"B3"];

 

        Return nil;

    }];

 

    RACSignal *zipSignal = [signalA zipWith:signalB];

 

    [zipSignal subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

             

              // output

              2017-01-03 13:48:09.234 ReactiveCocoa Advanced [3997:789720] zipWith: <RACTuple: 0x600000004df0> (

    A1,

    B1

              )

              2017-01-03 13:48:09.234 ReactiveCocoa Advanced [3997:789720] zipWith: <RACTuple: 0x608000003410> (

    A2,

    B2

              )

              ```

             

             

###### combineLatest

- ** role **

             

              Combine multiple signals and get the last value of each signal . Each merged signal must have sendNext at least once to trigger the merged signal.

 

- ** Bottom implementation **

             

              1. When the combined signal is subscribed, the internal will automatically subscribe to signalA , signalB, and both signals must be sent to be triggered.

              2. Combine the last transmitted values ​​of the two signals into a tuple.

                  

- ** Use **

 

              ```

              RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"A1"];

        [subscriber sendNext:@"A2"];

 

        Return nil;

    }];

 

    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"B1"];

        [subscriber sendNext:@"B2"];

        [subscriber sendNext:@"B3"];

 

        Return nil;

    }];

 

    RACSignal *combineSianal = [signalA combineLatestWith:signalB];

 

    [combineSianal subscribeNext:^(id x) {

 

        NSLog(@"combineLatest:%@", x);

    }];

             

              // output

              2017-01-03 13:48:09.235 ReactiveCocoa Advanced [3997:789720] combineLatest:<RACTuple: 0x60800000e150> (

    A2,

    B1

              )

              2017-01-03 13:48:09.235 ReactiveCocoa Advanced [3997:789720] combineLatest:<RACTuple: 0x600000004db0> (

    A2,

    B2

              )

              2017-01-03 13:48:09.236 ReactiveCocoa Advanced [3997:789720] combineLatest:<RACTuple: 0x60800000e180> (

    A2,

    B3

              )

              ```

             

- ** Note **

 

              **combineLatest** is similar to **zip** , and each merged signal must have at least one sendNext to trigger the merged signal.

             

              The difference is shown in the figure below:

             

              ![](https://ww2.sinaimg.cn/large/006y8lVagw1fbdf6cyez6j30id0kkabf.jpg)

 

 

###### reduce  

 

- ** role **

             

              Aggregate the value of the signal issuing tuple into a value

- ** Bottom implementation **

             

              1. Subscribe to the aggregated signal,

              2. Each time there is content sent, it will execute reduceblcok and convert the signal content into the value returned by reduceblcok .

                  

- ** Use **

 

     Common usage, (combined in aggregation first) `combineLatest:(id<NSFastEnumeration>)signals reduce:(id (^)())reduceBlock`

 

Introduction to the block in      reduce :

 

The parameters in      reduceblcok , how many combinations of signals, how many parameters does reduceblcok have , each parameter is the content of the previous signal

     The return value of reduceblcok : the content after the aggregated signal.

 

 

 

              ```

                  RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"A1"];

        [subscriber sendNext:@"A2"];

 

        Return nil;

    }];

 

    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@"B1"];

        [subscriber sendNext:@"B2"];

        [subscriber sendNext:@"B3"];

 

        Return nil;

    }];

 

 

    RACSignal *reduceSignal = [RACSignal combineLatest:@[signalA, signalB] reduce:^id(NSString *str1, NSString *str2){

 

        Return [NSString stringWithFormat:@"%@ %@", str1, str2];

    }];

 

    [reduceSignal subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    // output

    2017-01-03 15:42:41.803 ReactiveCocoa advanced [4248:1264674] A2 B1

              2017-01-03 15:42:41.803 ReactiveCocoa advanced [4248:1264674] A2 B2

              2017-01-03 15:42:41.803 ReactiveCocoa advanced [4248:1264674] A2 B3

 

              ```

             

#### 过滤

 

Filtering is the filtering of specific values in a signal , or filtering of a signal specifying the number of transmissions.

 

###### filter

 

- ** role **

 

              Filter the signal and use it to get the signal that meets the condition .

             

              The return value of the block is the Bool value, and the signal is filtered by returning `NO`.

             

- ** Use **

 

              ```

              // Filter :

              / / Each time the signal is sent, the filter condition will be judged first .

              [[_textField.rac_textSignal filter:^BOOL(NSString *value) {

 

        NSLog(@" 原信号: %@", value);

 

        / / Filter the signal length <= 3

        Return value.length > 3;

 

    }] subscribeNext:^(id x) {

 

        NSLog (@" signal longer than 3 : %@", x);

    }];

 

    / / Output 12345 in _textField

              // output

              2017-01-03 16:36:54.938 ReactiveCocoa advanced [4714:1552910] original signal : 1

              2017-01-03 16:36:55.383 ReactiveCocoa advanced [4714:1552910] original signal : 12

              2017-01-03 16:36:55.706 ReactiveCocoa advanced [4714:1552910] original signal : 123

              2017-01-03 16:36:56.842 ReactiveCocoa advanced [4714:1552910] original signal : 1234

              2017-01-03 16:36:56.842 ReactiveCocoa Advanced [4714:1552910] Signals longer than 3 : 1234

              2017-01-03 16:36:58.350 ReactiveCocoa advanced [4714:1552910] original signal : 12345

              2017-01-03 16:36:58.351 ReactiveCocoa Advanced [4714:1552910] Signals longer than 3 : 12345

              ```

             

###### ignore

 

- ** role **

 

              Ignore some signals .

             

- ** Use **

 

- ** role **

 

              Ignore signals with certain values .

             

              The underlying layer calls `filter` to compare with the filtered value. If it returns equal, `NO`

             

- ** Use **

 

              ```

                / / Internally call filter filtering, ignore the value of the character @ "1"

[[_textField.rac_textSignal ignore:@"1"] subscribeNext:^(id x) {

 

              NSLog(@"%@",x);

}];

 

 

              ```

 

###### distinctUntilChanged

 

- ** role **

 

              A signal is sent when there is a significant change in the last value and the current value, otherwise it will be ignored.

             

- ** Use **

 

              ```

              [[_textField.rac_textSignal distinctUntilChanged] subscribeNext:^(id x) {

 

        NSLog(@"%@",x);

    }];

              ```

             

###### skip             

 

- ** role **

 

              Skip ** of N times ** signal is transmitted .

             

- ** Use **

             

              ```

/ / indicates that the input is the first time, will not be monitored, skip the first signal sent

[[_textField.rac_textSignal skip:1] subscribeNext:^(id x) {

 

   NSLog(@"%@",x);

}];

              ```

 

 

 

##### take

- ** role **

 

              Take ** before N times ** signal is transmitted .

- ** Use **

 

              ```

              RACSubject *subject = [RACSubject subject] ;

 

    / / Take the signal sent twice before

    [[subject take:2] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    [subject sendNext:@1];

    [subject sendNext:@2];

    [subject sendNext:@3];

 

    // output

              2017-01-03 17:35:54.566 ReactiveCocoa advanced [4969:1677908] 1

              2017-01-03 17:35:54.567 ReactiveCocoa advanced [4969:1677908] 2

              ```

 

###### takeLast

 

- ** role **

 

              Take ** last N times ** signals transmitted

             

              Prerequisites, the subscriber must call completion `sendCompleted` , because only the completion, you know how many signals there are .

             

- ** Use **             

 

              ```

              RACSubject *subject = [RACSubject subject] ;

 

    / / Take the signal sent twice

    [[subject takeLast:2] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    [subject sendNext:@1];

    [subject sendNext:@2];

    [subject sendNext:@3];

 

    / / must jump to complete

    [subject sendCompleted];

              ```

 

###### takeUntil

 

- ** role **

 

              Acquire the signal until a signal is executed

- ** Use **             

 

              ```

              / / Listen to the text box changes until the current object is destroyed

[_textField.rac_textSignal takeUntil:self.rac_willDeallocSignal];

              ```

             

###### switchToLatest

- ** role **

 

              For signalOfSignals (signal signals), sometimes the signal will also signal, in signalOfSignals , get the latest signal sent by signalOfSignals .

             

- ** Note **

 

              switchToLatest : can only be used for signals in the signal

 

- ** Use **             

 

              ```

              RACSubject *signalOfSignals = [RACSubject subject];

    RACSubject *signal = [RACSubject subject];

 

    / / Get the signal in the signal recently issued a signal, subscribe to the most recent signal.

    [signalOfSignals.switchToLatest subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    [signalOfSignals sendNext:signal];

    [signal sendNext:@1];

              ```

 

#### Order

 

The order includes the two methods `doNext` and `doCompleted` , mainly to execute the Block in these methods before executing `sendNext` or `sendCompleted` .

 

###### doNext

             

Before executing `sendNext` , this block of `doNext` will be executed first .

 

###### doCompleted

 

Perform `sendCompleted` before, will first execute this ` doCompleted` of `Block`

 

```

[[[[Signature createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

    [subscriber sendNext:@"hi"];

 

    [subscriber sendCompleted];

 

    Return nil;

 

}] doNext:^(id x) {

 

    // This block will be called before executing [subscriber sendNext:@"hi"]

    NSLog(@"doNext");

 

}] doCompleted:^{

 

    // This block will be called before executing [subscriber sendCompleted]

    NSLog(@"doCompleted");

}] subscribeNext:^(id x) {

 

    NSLog(@"%@", x);

}];

 

 

```

 

#### thread

 

** ReactiveCocoa ** in thread operations include `deliverOn` and ` subscribeOn` both the * content delivery * or when creating a signal * block code * switch to the specified thread execution.

 

###### deliverOn

 

- ** role **

 

              The content delivery is switched to the development thread, the side effect is in the original thread, and the code in the block when the signal is created is called a side effect.

- ** Use **

 

              ```

              / / Execute in the child thread

              Dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

 

        [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

            NSLog(@"%@", [NSThread currentThread]);

 

            [subscriber sendNext:@123];

 

            [subscriber sendCompleted];

 

            Return nil;

        }]

          deliverOn:[RACScheduler mainThreadScheduler]]

 

         subscribeNext:^(id x) {

 

             NSLog(@"%@", x);

 

             NSLog(@"%@", [NSThread currentThread]);

         }];

    });

 

    // output

2017-01-04 10:35:55.415 ReactiveCocoa Advanced [1183:224535] <NSThread: 0x608000270f00>{number = 3, name = (null)}

2017-01-04 10:35:55.415 ReactiveCocoa advanced [1183:224482] 123

2017-01-04 10:35:55.415 ReactiveCocoa Advanced [1183:224482] <NSThread: 0x600000079bc0>{number = 1, name = main}

              ```

             

              You can see that ` side effects` are executed in * subthread * , and ` delivered content` is received in * main thread *

 

 

###### subscribeOn

- ** role **

 

              ** subscribeOn ** sucked ` content delivery ' and ` side ' will switch to the specified thread

- ** Use **

 

              ```

    Dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

 

        [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

            NSLog(@"%@", [NSThread currentThread]);

 

            [subscriber sendNext:@123];

 

            [subscriber sendCompleted];

 

            Return nil;

        }]

          subscribeOn:[RACScheduler mainThreadScheduler]] // delivered content to the main thread

         subscribeNext:^(id x) {

 

             NSLog(@"%@", x);

 

             NSLog(@"%@", [NSThread currentThread]);

         }];

    });             

              //

2017-01-04 10:44:47.558 ReactiveCocoa Advanced [1243:275126] <NSThread: 0x608000077640>{number = 1, name = main}

2017-01-04 10:44:47.558 ReactiveCocoa advanced [1243:275126] 123

2017-01-04 10:44:47.558 ReactiveCocoa Advanced [1243:275126] <NSThread: 0x608000077640>{number = 1, name = main}

              ```

             

              ` Content delivery ` and ` side effects ' are switched to the * main thread * execution

             

#### 时间

 

The time operation sets the signal timeout, timing and delay.

 

###### interval timing

- ** role **

 

              Timing: Signal every once in a while

             

              ```

              / / Send a signal every 1 second, specify the current thread to execute

              [[RACSignal interval:1 onScheduler:[RACScheduler currentScheduler]] subscribeNext:^(id x) {

 

        NSLog (@" Timing : %@", x);

    }];

 

              // output

              2017-01-04 13:48:55.196 ReactiveCocoa advanced [1980: 492724] Timing : 2017-01-04 05:48:55 +0000

              2017-01-04 13:48:56.195 ReactiveCocoa advanced [1980: 492724] Timing : 2017-01-04 05:48:56 +0000

              2017-01-04 13:48:57.196 ReactiveCocoa advanced [1980: 492724] Timing : 2017-01-04 05:48:57 +0000

              ```

 

 

###### timeout timeout

 

- ** role **

 

              Timeout allows a signal to automatically report an error after a certain amount of time.

             

              ```

              RACSignal *signal = [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        / / Do not send a signal, simulate the timeout state

        // [subscriber sendNext:@"hello"];

        //[subscriber sendCompleted];

 

        Return nil;

    }] timeout: 1 onScheduler: [RACScheduler currentScheduler]]; / / set 1 second timeout

 

    [signal subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    } error:^(NSError *error) {

 

        NSLog(@"%@", error);

    }];

 

    / / Execute the code after 1 second output:

    2017-01-04 13:48:55.195 ReactiveCocoa Advanced [1980: 492724] Error Domain=RACSignalErrorDomain Code=1 "(null)"

              ```

 

###### delay delay

- ** role **

 

              Delay, send a signal after a delay

             

              ```

              RACSignal *signal2 = [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext: @" delay output "];

 

        Return nil;

    }] delay:2] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    / / Execute the code after 2 seconds output

    2017-01-04 13:55:23.751 ReactiveCocoa Advanced [2030:525038] Delayed Output

              ```

 

 

#### 重复

 

###### retry

 

- ** role **

 

              Retry: As long as the error `sendError:` is sent , the block that created the signal will be re-executed until it succeeds.

             

              ```

              __block int i = 0;

 

    [[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        If (i == 5) {

 

            [subscriber sendNext:@"Hello"];

 

        } else {

 

            // Send error

            NSLog (@" received error : %d", i);

            [subscriber sendError:nil];

        }

 

        i++;

 

        Return nil;

 

    }] retry] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

 

    } error:^(NSError *error) {

 

        NSLog(@"%@", error);

 

    }];

 

              // output

2017-01-04 14:36:51.594 ReactiveCocoa Advanced [2443:667226] received the error message : 0

2017-01-04 14:36:51.595 ReactiveCocoa Advanced [2443:667226] received the error message : 1

2017-01-04 14:36:51.595 ReactiveCocoa Advanced [2443:667226] received the error message : 2

2017-01-04 14:36:51.596 ReactiveCocoa Advanced [2443:667226] received the error message : 3

2017-01-04 14:36:51.596 ReactiveCocoa Advanced [2443:667226] received the error message : 4

2017-01-04 14:36:51.596 ReactiveCocoa advanced [2443:667226] Hello

 

              ```

 

###### replay

 

- ** role **

 

              Replay: When a signal is subscribed multiple times , the content is played repeatedly

             

              ```

              RACSignal *signal = [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

        [subscriber sendNext:@1];

        [subscriber sendNext:@2];

 

        Return nil;

    }] replay];

 

    [signal subscribeNext:^(id x) {

        NSLog(@"%@", x);

    }];

 

    [signal subscribeNext:^(id x) {

        NSLog(@"%@", x);

    }];

 

    // output

2017-01-04 14:51:01.934 ReactiveCocoa advanced [2544:706740] 1

2017-01-04 14:51:01.934 ReactiveCocoa advanced [2544:706740] 2

2017-01-04 14:51:01.934 ReactiveCocoa advanced [2544:706740] 1

2017-01-04 14:51:01.935 ReactiveCocoa advanced [2544:706740] 2

              ```

 

 

###### throttle

 

- ** role **

 

              Throttling : When a signal is sent more frequently, throttling can be used. The signal content is not sent for a certain period of time, and the latest content of the signal is sent after a period of time.

             

              ```

              RACSubject *subject = [RACSubject subject];

 

    // throttle for 1 second, receive the last transmitted signal after 1 second

    [[subject throttle:1] subscribeNext:^(id x) {

 

        NSLog(@"%@", x);

    }];

 

    [subject sendNext:@1];

    [subject sendNext:@2];

    [subject sendNext:@3];

 

    // output

    2017-01-04 15:02:37.543 ReactiveCocoa advanced [2731:758193] 3

              ```

 

# MVVM Architecture Thoughts

---

Why should the program have an architecture? Easy to program development and maintenance .

 

#### Common architecture

- **MVC**

 

              M: Model V: View C: Controller

 

- **MVVM**

 

              M: Model V: View + Controller VM: View Model

 

- **MVCS**

 

              M: Model V: View C: Controller C: Service Class

 

- [**VIPER**] (http://www.cocoachina.com/ios/20140703/9016.html)

 

              V: View I: Interactor P: Presenter E: Entity R: Route

 

#### Introduction to MVVM

 

- Model (M): Save view data.

 

- View + Controller (V): Show content + How to display

 

- View Model (VM): Processes the business logic of the presentation, including button clicks, data requests and parsing, and more.

 

# 实战一: Login interface

 

#### demand

1. Listen to the contents of two text boxes

2. There is a content login button to allow button clicks

3. Return to the login result

 

#### 分析

1. All business logic of the interface is handed over to the controller for processing.

2. In the MVVM architecture, all the controller services are moved to the VM model, that is, each controller corresponds to a VM model .

 

#### Steps

1. Create a LoginViewModel class to handle the login interface business logic .

2. This class should store the account information, create an account Account model

3. The LoginViewModel should hold the account information Account model.

4. Need to monitor the account and password changes in the Account model at all times, how to monitor?

5. In non- RAC development, it is a habit assignment. In RAC development, you need to change the development thinking. From assignment to binding, you can bind the properties in the Account model at the beginning of initialization. Need to override the set method.

6. Each time the value of the Account model changes, you need to determine whether the button can be clicked, and do the processing in the VM model to provide the outside world with a signal to click the button .

7. The registration signal requires judgment Account in the account and password if there is value, with KVO monitor changes in these two values, their aggregation into a registration signal .

8. The click of the monitor button, processed by the VM , should declare a RACCommand to the VM , specifically handling the login business logic .

9. Execute the command to wrap the data into a signal

10. Monitor the data transfer of the signal in the command

11. Listening command execution time

 

 

 

#### Running effect

 

![ Login interface ](https://ww3.sinaimg.cn/large/006y8lVagw1fbgvoh8yu6j30bj0l43yz.jpg)

 

#### Code

 

`MyViewController.m`

 

```

#import "MyViewController.h"

#import "LoginViewModel.h"

 

@interface MyViewController ()

 

@property (nonatomic, strong) LoginViewModel *loginViewModel;

 

@property (weak, nonatomic) IBOutlet UITextField *accountField;

 

@property (weak, nonatomic) IBOutlet UITextField *pwdField;

 

@property (weak, nonatomic) IBOutlet UIButton *loginBtn;

 

@end

 

@implementation MyViewController

 

- (void)viewDidLoad {

    [super viewDidLoad];

 

    [self bindModel];

 

}

 

- (void)didReceiveMemoryWarning {

    [super didReceiveMemoryWarning];

    // Dispose of any resources that can be recreated.

}

 

 

 

/ / View model binding

- (void)bindModel {

 

    / / Bind the signal to the model's properties

    //

    RAC(self.loginViewModel.account, account) = _accountField.rac_textSignal;

    RAC(self.loginViewModel.account, pwd) = _pwdField.rac_textSignal;

 

    RAC(self.loginBtn, enabled) = self.loginViewModel.enableLoginSignal;

 

    / / Monitor login click

    [[_loginBtn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {

 

        [self.loginViewModel.LoginCommand execute:nil];

    }];

 

}

- (IBAction)btnTap:(id)sender {

 

 

}

 

#pragma mark - lazyLoad

 

- (LoginViewModel *)loginViewModel {

 

    If (nil == _loginViewModel) {

        _loginViewModel = [[LoginViewModel alloc] init];

    }

 

    Return _loginViewModel;

}

```             

                           

`LoginViewModel.h`

 

```

#import <UIKit/UIKit.h>

 

@interface Account : NSObject

 

@property (nonatomic, strong) NSString *account;

@property (nonatomic, strong) NSString *pwd;

 

@end

 

 

@interface LoginViewModel : UIViewController

 

@property (nonatomic, strong) Account *account;

 

/ / Whether to allow the login signal

@property (nonatomic, strong, readonly) RACSignal *enableLoginSignal;

 

@property (nonatomic, strong, readonly) RACCommand *LoginCommand;

 

@end

 

```

 

`LoginViewModel.m`

 

```

#import "LoginViewModel.h"

 

@implementation Account

 

@end

 

 

@interface LoginViewModel ()

 

@end

 

@implementation LoginViewModel

 

- (instancetype)init {

 

    If (self = [super init]) {

        [self initialBind];

    }

    Return self;

}

 

- (void)initialBind {

 

    / / Listen to the account attribute changes, synthesize them a signal

    _enableLoginSignal = [RACSubject combineLatest:@[RACObserve(self.account, account), RACObserve(self.account, pwd)] reduce:^id(NSString *accout, NSString *pwd){

 

        Return @(accout.length && pwd.length);

    }];

 

    / / Processing business logic

    _LoginCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {

 

        NSLog (@" Clicked Login ");

        Return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

            / / Imitation of network latency

 

            Dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{

 

                / / Return to the login successfully sent success signal

                [subscriber sendNext:@" Login successful "];

            });

 

            Return nil;

        }];

    }];

 

 

    / / Monitor the data generated by the login

    [_LoginCommand.executionSignals.switchToLatest subscribeNext:^(id x) {

 

        If ([x isEqualToString:@" Login succeeded "]) {

            NSLog (@" login success ");

        }

 

    }];

 

    [[_LoginCommand.executing skip:1] subscribeNext:^(id x) {

 

        If ([x isEqualToNumber:@(YES)]) {

 

            NSLog (@" is landing ...");

        } else {

 

        / / Login successfully

        NSLog (@" Login Success ");

 

        }

 

    }];

}

 

#pragma mark - lazyLoad

 

- (Account *)account

{

    If (_account == nil) {

        _account = [[Account alloc] init];

    }

    Return _account;

}

 

- (void)viewDidLoad {

    [super viewDidLoad];

 

}

 

@end

 

```

 

# 实战二: Network request data

 

#### demand

1. requested data segment of a network, the request data in `tableView` appear on

2. The data is the search result of the Douban book, URL : url: https://api.douban.com/v2/book/search?q=Wukong Chuan

 

#### 分析

1. All business logic of the interface is handed over to the ** controller ** for processing

2. The network request is handed over to the **MV** model processing

 

#### Steps

 

1. The controller provides a view model ( requesViewModel ) that handles the business logic of the interface

2. The VM provides a command to process the request business logic

3. In the block that creates the command , the request is wrapped into a signal, and when the request is successful, the data is passed out.

4. Request data success, the dictionary should be converted into a model, saved to the view model, the controller wants to use it directly from the view model.

 

#### 其他

 

The network request and image cache use [AFNetworking] (https://github.com/AFNetworking/AFNetworking) and [SDWebImage] (https://github.com/rs/SDWebImage), and import them in Pods .

 

```

Platform :ios, '8.0'

 

Target 'ReactiveCocoa Advanced ' do

 

Use_frameworks!

Pod 'ReactiveCocoa', '~> 2.5'

Pod 'AFNetworking'

Pod 'SDWebImage'

End

```

 

#### Running effect

 

![](https://ww3.sinaimg.cn/large/006y8lVagw1fbgw1xnz74j30bj0l4408.jpg)

 

 

#### Code

 

`SearchViewController.m`

 

```

#import "SearchViewController.h"

#import "RequestViewModel.h"

 

@interface SearchViewController ()<UITableViewDataSource>

 

@property (nonatomic, strong) UITableView *tableView;

 

@property (nonatomic, strong) RequestViewModel *requesViewModel;

 

@end

 

@implementation SearchViewController

 

- (RequestViewModel *)requesViewModel

{

    If (_requesViewModel == nil) {

        _requesViewModel = [[RequestViewModel alloc] init];

    }

    Return _requesViewModel;

}

 

- (void)viewDidLoad {

    [super viewDidLoad];

 

 

    self.tableView = [[UITableView alloc] initWithFrame:self.view.frame];

 

    self.tableView.dataSource = self;

 

    [self.view addSubview:self.tableView];

 

    //

    RACSignal *requesSiganl = [self.requesViewModel.reuqesCommand execute:nil];

 

    [requesSiganl subscribeNext:^(NSArray *x) {

 

        self.requesViewModel.models = x;

 

        [self.tableView reloadData];

    }];

}

 

- (void)didReceiveMemoryWarning {

    [super didReceiveMemoryWarning];

    // Dispose of any resources that can be recreated.

}

 

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section

{

    Return self.requesViewModel.models.count;

}

 

- (UITableViewCell *) tableView: (UITableView *) tableView cellForRowAtIndexPath: (NSIndexPath *) indexPath

{

    Static NSString *ID = @"cell";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    If (cell == nil) {

 

        Cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];

    }

 

    Book *book = self.requesViewModel.models[indexPath.row];

    cell.detailTextLabel.text = book.subtitle;

    cell.textLabel.text = book.title;

 

    [cell.imageView sd_setImageWithURL:[NSURL URLWithString:book.image] placeholderImage:[UIImage imageNamed:@"cellImage"]];

 

 

    Return cell;

}

@end

```

 

`RequestViewModel.h`

 

```

#import <Foundation/Foundation.h>

 

@interface Book : NSObject

 

@property (nonatomic, copy) NSString *subtitle;

@property (nonatomic, copy) NSString *title;

@property (nonatomic, copy) NSString *image;

 

@end

 

@interface RequestViewModel : NSObject

 

/ / Request command

@property (nonatomic, strong, readonly) RACCommand *reuqesCommand;

 

/ / Model array

@property (nonatomic, strong) NSArray *models;

 

 

@end

```

 

`RequestViewModel.m`

 

```

#import "RequestViewModel.h"

 

@implementation Book

 

- (instancetype)initWithValue:(NSDictionary *)value {

 

    If (self = [super init]) {

 

        Self.title = value[@"title"];

        Self.subtitle = value[@"subtitle"];

        Self.image = value[@"image"];

    }

    Return self;

}

 

+ (Book *)bookWithDict:(NSDictionary *)value {

 

    Return [[self alloc] initWithValue:value];

}

 

 

 

@end

 

@implementation RequestViewModel

 

- (instancetype)init

{

    If (self = [super init]) {

 

        [self initialBind];

    }

    Return self;

}

 

 

- (void)initialBind

{

    _reuqesCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {

 

      RACSignal *requestSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

 

          NSMutableDictionary *parameters = [NSMutableDictionary dictionary];

          Parameters[@"q"] = @" 悟空传";

 

          //

          [[AFHTTPSessionManager manager] GET:@"https://api.douban.com/v2/book/search" parameters:parameters progress:^(NSProgress * _Nonnull downloadProgress) {

 

              NSLog(@"downloadProgress: %@", downloadProgress);

          } success:^(NSURLSessionDataTask * _Nonnull task, id _Nullable responseObject) {

 

              / / Data request is successful, the data is sent out

              NSLog(@"responseObject:%@", responseObject);

 

              [subscriber sendNext:responseObject];

 

              [subscriber sendCompleted];

 

          } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {

 

              NSLog(@"error: %@", error);

          }];

 

 

         Return nil;

      }];

 

        / / When returning the data signal, the dictionary in the data is mapped into a model signal, passed out

        Return [requestSigan l map:^id(NSDictionary *value) {

 

            NSMutableArray *dictArr = value[@"books"];

 

            NSArray *modelArr = [[dictArr.rac_sequence map:^id(id value) {

 

                Return [Book bookWithDict:value];

 

            }] array];

 

            Return modelArr;

 

        }];

 

    }];

}

 

 

@end

 

```
