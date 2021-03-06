基礎知識
--------

在進入如何使用蘋果的 API 之前，先講一些跟 Audio 相關的基礎知識。這些資
料在 wikipedia 上都可以查到，我們盡可能簡短說明。

### 聲音與數位音訊

聲音是空氣或是在水中的震動，是一種能夠被人耳所感知的類比訊號，單位是分貝（dB），
人耳可以忍受的聲音介於 0 到 100 分貝之間。震動幅度的強弱決定了是大聲或是小聲，震
動頻率的密集程度決定了是高音或是低音。

所謂的數位音訊或數位音樂，就是將聲波這種連續的類比訊號，轉換成一連串的數字，用零
與一的二進位數字盡可能地重現在大自然中發生的事物；用一個數字，標示在某個時間單位
時訊號的強弱程度。

之所以說「盡可能」，是因為類比轉換成數位的過程中一定會有所捨棄。我們捨棄了超過
100 分貝的部份，在大自然中存在超過 100 分貝的聲音，但是超過了人耳可以忍受的範
圍；而在大自然中，時間可以不斷無限切分到人類感官無法感知的單位，但我們要將訊號記
錄下來，還是必須要定義一個單位。這種將一個一個時間單位的訊號強弱程度記錄下來的過
程，叫做採樣（sampling），每一個時間單位上的樣本，叫做 sample，或是一個 frame，
一秒鐘內有多少 sample，叫做採樣比（sample rate）。

現在定義的 CD 音質是 44100 Hz，也就是一秒鐘內有 44100 個 sample，這個規格的原因
是人耳可以聽到的頻率介於 20 到 20000 Hz 之間，根據 Nyquist–Shannon 的採樣定律，只
要採樣比超過 20000 Hz 的兩倍，人耳就根本聽不出差別。被錄製下來的聲音可以用帶號、
非帶號的整數或浮點數儲存，如果是非帶號整數的話，每個 sample 會介於 0 到整數最大
值之間，帶號整數就可能是整數最大值到最小值之間，非帶號浮點數則會介於 0.0 到 1.0
之間，帶號浮點數就可能會在 -1 到 1 之間。

這種完全還沒有壓縮過的資料，就叫做 PCM （Pulse-code modulation）格式（請參考
https://en.wikipedia.org/wiki/Pulse-code_modulation ），我們常用的WAV、AIFF、CAF
檔案，都是這種格式，而 Audio 硬體最後要播放的，也是這種資料。所以，CD其實就是一
秒鐘有44100 個 16 位元帶正負號的整數，而從 CD 開始，其實我們就已經進入了數位音樂
時代。

沒有壓縮過的音訊檔案以現在的眼光看都還是很大。比方說，我們用 16 位元非帶號整數
（兩個 byte）儲存，加上左右聲道，一首歌曲就要 30 mb（44100 × 2 × 2 × 60 × 3 ÷
1024 ÷ 1024），雖然可以存放在 CD 這種介質上，但並不適合網路傳輸。到了 90 年代，
MP3 等壓縮格式開始流行，音樂變得可以在網路上傳輸、交換，從此也大幅改變了音樂產
業，人們從購買 CD 變成透過網路取得音樂，也讓現在可以有 KKBOX 這樣的網路串流音樂
服務。

### 壓縮音檔：Codec 與 Container

壓縮過的音訊檔案尺寸大大降低，但像 MP3 等格式是破壞性的壓縮，在壓縮過程中，也會
造成音質的降低與失真，所以壓縮的比例同時影響壓縮後的大小與聆聽時感受到的品質。

音檔壓縮的比例我們使用 bitrate 描述，bitrate 就是這個音檔用多少的資料（注意，單
位是 bit，不是 byte）表示一秒鐘的聲音，KKBOX 目前提供 128kb、192kb、320kb 三種不
同品質的音檔，單位都是 bitrate。kb 是使用十進位，所以我們可以推算出一首三分鐘的
320kb 的音檔，大約 6.8 MB（320 / 8 * 1000 * 60 * 3 / 1024 / 1024）。

當我們將原始音檔壓縮成 MP3 格式的時候，並不是把整個檔案都壓縮起來，而是先將連續
的原始音檔切成一個一個的小塊，然後一次只壓縮一個小塊，這樣的小塊叫做 packet，MP3
格式的每個 packet 為 1152 個 frame，用來將每個小塊做壓縮與解壓縮的程式就是
codec。

之後，我們會在每個 packet 的前方都加上一個簡短的檔頭，標示在這之後是一個
packet，以及 packet 的長度，一個 MP3 檔案，就是連續的檔頭與 packet的集合，我們要
播放 MP3 檔案，首先就是要透過 parser，找出每個 packet 所在的位置。AAC ADTS 格式
也是使用這種方式包裝資料。

至於 MP4 檔案則是用另外一種方式包裝資料。不同於 MP3 是連續的檔頭與packet，MP4 格
式則是一種巢狀結構：在一個 MP4 檔案中，會有許多叫做 atom的容器，一個 atom 裡頭可
能還會有其他的 atom。其中最主要的兩個 atom 分別是 moov 與 mdat，mdat 裡頭是連續
的 packet 資料，但是這一段資料中並不會特別著名哪一個 packet 從哪裡開始到結束，而
moov 這邊有所有的 packet的 offset 位置，也就是，mdat 裡頭有哪些 packet，要拿
moov 這段資料當做索引。用來包裝 packet 的格式，就叫做 container。

moov 與 mdat 不一定要哪個在前哪個在後，但如果你要使用 Core Audio API播放 MP4 檔
案，moov 一定要放在 mdat 前方，不然 Core Audio API 所提供的格式 parser 會告訴
你：它不支援沒有 optimized 過的 MP4 檔案。

但很奇妙，如果你把這個檔案拿去 iTunes 或 QuickTime 裡頭，卻可以正常播放；所以
呢，其實蘋果自己的播放軟體裡頭用的底層，與蘋果公開的 API 並不是相同的東西，我們
也不能期待用蘋果的播放軟體能夠播的檔案，我們就能夠播出來。如果想要知道某個檔案能
不能用蘋果的公開 API 播放，可以使用 command line 底下的 afinfo 與 afplay 指令檢
查。

當然，如果你不信任系統提供的 parser，也可以寫自己的 parser。iOS 與 Mac OS X 上的
audio format parser 叫做 Audio File Stream Service，在 Mac上是 10.5 以後才出現的
API；KKBOX Mac 版是在 2008 年開始開發，當時還必須支援 10.4，因此我們最早也寫了自
己的 MP3 parser。

所以，當我們在稱呼一種音檔的時候，通常得要同時說明這種音檔的 codec 與 Container：

* MP3 同時是一種 codec 與 container，所以可以直接稱呼成 MP3
* AAC 是一種 codec，但是有 MP4 與 ADTS 這幾種不同的 container 格式，所以我們會稱
  為 AAC MP4 或是 AAC ADTS。
* OGG 是一種 container，裡頭用的 codec 通常是 Vorbis，但 FLAC 格式雖然平常放在
  FLAC 自己的 container 中，但也有可能用 OGG container 格式包裝 FLAC 資料，這種
  格式我們就會稱為 OGG FLAC。

### 播放網路串流的流程

所以，如果現在我們想要播放位在 server 上的某個 MP3 或 AAC 音檔，流程大
概就是：

1. 發送網路連線，跟 server 要求讀取這個檔案，並且等候連線的 callback
2. 在連線的 callback 中儲存資料，如果這個檔案經過 DRM 保護的話，也要在
   這時候做加解密
3. 把收到的資料送到 parser 中，parse 出檔案格式與 packet 的位置
4. 保存這些 packet
5. 按照播放硬體的播放進度，定時把這些 packet 交給 converter，讓
   converter 呼叫 codec，將我們的 packet 轉換成 PCM 格式
6. 讓 Audio 硬體播放 PCM 格式的資料

![流程圖](flow.png)
