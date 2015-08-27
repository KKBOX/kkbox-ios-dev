實作 NSCoding
-------------

NSCoding 是一個 protocol，裡頭只有兩個 method 要實作

``` objc
@protocol NSCoding
- (void)encodeWithCoder:(NSCoder *)aCoder;
- (id)initWithCoder:(NSCoder *)aDecoder;
@end
```

`encodeWithCoder:` 的用途是將我們的物件透過 NSCoder 轉成 NSData，至於
`initWithCoder:` 剛好相反，就是透過 NSCoder，再把 NSData 轉回物件。

假如我們現在有一個叫做 KKSongTrack 的 class，用來代表 KKBOX 裡頭的一首
歌，裡頭只有 songName、albumName 與 artistName 三個 property，這三個
property 都是 NSString。

``` objc
@interface KKSongTrack : NSObject <NSCoding>
@property (strong, nonatomic) NSString *songName;
@property (strong, nonatomic) NSString *albumName;
@property (strong, nonatomic) NSString *artistName;
@end
```

在實作的時候，我們就可以用 NSCoder 的相關 method 對資料做 encode 與
decode。由於我們的 property 都是 NSString，都是 Objective-C 物件，所以
我們選擇 `encodeObject:forKey:` 以及 `decodeObjectForKey:`。實作如下：

``` objc
#import "KKSongTrack.h"

static NSString *const kSongNameKey = @"song_name";
static NSString *const kAlbumNameKey = @"album_name";
static NSString *const kArtistNameKey = @"artist_name";

@implementation KKSongTrack

- (NSString *)description
{
  return [NSString stringWithFormat:@"<%@ %p %@ - %@ - %@>",
    NSStringFromClass([self class]), self,
	self.songName, self.albumName, self.artistName];
}

- (instancetype)initWithCoder:(NSCoder *)coder
{
	self = [super init];
	if (self) {
		self.songName = [coder decodeObjectForKey:kSongNameKey];
		self.albumName = [coder decodeObjectForKey:kAlbumNameKey];
		self.artistName = [coder decodeObjectForKey:kArtistNameKey];
	}
	return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder
{
	[aCoder encodeObject:self.songName forKey:kSongNameKey];
	[aCoder encodeObject:self.albumName forKey:kAlbumNameKey];
	[aCoder encodeObject:self.artistName forKey:kArtistNameKey];
}

@end
```

除了 Objective-C 物件型別外，NSCoder 還有處理 BOOL、整數以及浮點數相關
的 method 可以使用。

接下來如果我們想把歌曲轉成 NSData：

``` objc
KKSongTrack *song = [[KKSongTrack alloc] init];
song.songName = @"orz 之歌";
song.albumName = @"orz 專輯";
song.artistName = @"orz";
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:song];
```

把 NSData 再轉回 KKSongTrack：

```
KKSongTrack *decodeSong = [NSKeyedUnarchiver unarchiveObjectWithData:data];
```
