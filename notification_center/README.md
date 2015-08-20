Notification Center
===================

Notification Center 是在 Cocoa/Cocoa Touch Framework 中，物件之間可以
不必互相知道彼此的存在，也可以互相交換資料/狀態的機制。

我們可以把 Notification Center 想像是一種廣播系統。當一個物件 A 的狀態
發生改變，而有多個物件需要知道這個物件發生改變的狀況下，物件 A 不必直接
對這些物件發出呼叫，而是告訴一個廣播中心說：「我的狀態改變了」，至於其
他需要聽取狀態的物件呢，也只要對這個廣播中心訂閱（subscribe）指定的通知，
所以當物件 A 發出通知的時候，這個廣播中心就會通知有訂閱通知的其他物件。
這個廣播中心，就是 Notification Center。

我們經常使用 Notification Center 處理來自作業系統的事件。假如我們現在
寫了一個日記軟體，這個軟體裡頭已經有很多 view，每個 view 裡頭都有一篇
日記，每篇日記上都有該篇日記的撰寫日期與時間。我們通常會使用
NSDateFormatter，使用系統偏好設定中的語系（Locale）設定，將日期轉成符
合語系設定的字串顯示，那麼，當用戶調整了系統偏好設定，像是把中文改成英
文，那麼，我們原本用中文顯示的日期，也應該馬上變成用英文顯示─我們該怎
麼做呢？

我們最常使用的通知中心是 NSNotificationCenter 這個 class，我們也通常使
用這個 class 的 singleton 物件 default center（也就是說，其實
Notification Center 有好幾個，不過我們最常使用的還是這個）。當系統語系
改變的時候，Notification Center 就會發出叫做
NSCurrentLocaleDidChangeNotification 的這項通知。所以，我們所有要顯示
日期的畫面物件，都應該要訂閱這個通知，在收到通知的時候，就要重新產生日
期字串。

接收 Notification
-----------------

一個通知分成幾個部分

1. **object**: 是誰送出了這個通知
2. **name**: 這個通知叫做什麼名字
3. **user info**: 這個通知還帶了哪些額外資訊

所以，當我們想要監聽某個通知的時候，便是指定要收聽由誰所發出、哪個名字
的通知，並且指定負責處理通知的 selector，以前面處理 locale 改變的例子來
看，我們就會寫出這樣的 code：

``` objc
- (void)viewDidLoad
{
	[super viewDidLoad];
	[[NSNotificationCenter defaultCenter] addObserver:self
	  selector:@selector(localeDidChange:)
	  name:NSCurrentLocaleDidChangeNotification
	  object:nil];
}

- (void)localeDidChange:(NSNotification *)notification
{
}

- (void)dealloc
{
	[[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

意思就是，我們要指定 name 為 NSCurrentLocaleDidChangeNotification 的通
知，交由 `localeDidChange:` 處理。在這邊 object 設定為 nil，代表不管是
哪個物件送出的，只要符合 NSCurrentLocaleDidChangeNotification 的通知，
我們統統都處理。

每個通知當中，還可能會有額外的資訊，會夾帶在 NSNotification 物件的
userInfo 屬性中，userInfo 是一個 NSDictionary。

像是我們如果讓某個 text field 變成了 first responder，那麼，在 iOS 上，
就會出現螢幕鍵盤，而當螢幕鍵盤出現的時候，我們往往要調整畫面的 layout，
而螢幕鍵盤在不同輸入法下的大小不一樣，像是跟英文鍵盤比較起來，中文輸入
鍵盤上面往往會有一塊組字區，而造成中文鍵盤比英文鍵盤大。在螢幕鍵盤升起
來的時候，我們會收到 UIKeyboardWillShowNotification 這項通知，而這項通
知就會用 userInfo 告訴我們鍵盤尺寸與位置。

如果我們在 `-addObserver:selector:name:object:` 裡頭，把 name 指定成
nil，就代表我們想要訂閱所有的通知，通常不太會有這種情境，不過有時候你想
要知道系統內部發生了什麼事情，可以用這種方式試試看。

當我們不需要繼續訂閱某項通知的時候，記得對 Notification Center 呼叫
`-removeObserver:`。我們通常在 dealloc 的時候停止訂閱。


發送 Notification
-----------------

Notification Queue
------------------

Notification 與 Threading
-------------------------

CFNotificationCenter
--------------------


Mac 上的其他 Notification Center
--------------------------------

### NSDistributedNotificationCenter

### NSWorkSpace 的 Notification Center


相關文章
--------

- [Notification Programming Topics](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html#//apple_ref/doc/uid/10000043i)