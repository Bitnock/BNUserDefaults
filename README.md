## BNUserDefaults

BNUserDefaults is a lightweight wrapper around `NSUserDefaults`. The only functionally significant that it does out of the box is register the defaults you provide when the class is initialized, so you don't need to do it yourself.

#### How to use

BNUserDefaults is intended to be subclassed. The subclass should provide two things: a dictionary of default values for properties within the defaults system, and, optionally, any maintenance code that should be run to keep certain values in the defaults system in sync with their actual value.

The dictionary of default values will be registered as the defaults system's defaults when the `BNUserDefaults` class is initialized. You could load these values from a plist if you prefer to keep them out of your code.

Your subclass is also a good place to collect methods that you may call manually for dealing with application settings and values.

For the sake of consistency, and also because the names of properties and method definitions on `NSUserDefaults` can be somewhat confusing, `BNUserDefauts` provides a getter called `settings`, which simply returns `[NSUserDefaults standardUserDefaults]`.

In practice you would do something like

	[ABUserDefaults.settings setObject:anObj forKey:AKey]

Be sure to call `[super initialize]` if you override the initialize method.

### Example

In this example, there will be an `incrementLaunchCount` that we expect to get called from other parts of the app, and a `cacheVersionString` that we only want to get called when the class is initialized. It stores a string value of the current app version, which could be displayed in the iOS Setting app.

**ABUserDefaults.h**

	#import "BNUserDefaults.h"
	
	extern NSString * const ABUserDefaultsKeyVersionString;
	extern NSString * const ABUserDefaultsKeyLaunchCount;
	extern NSString * const ABUserDefaultsKeyWWANDownloading;
	
	@interface ABUserDefaults : BNUserDefaults
	
	+ (int)incrementLaunchCount;
	
	@end

**ABUserDefaults.m**

	#import "ABUserDefaults.h"

	NSString * const ABUserDefaultsKeyVersionString = @"ABUserDefaultsKeyVersionString";
	NSString * const ABUserDefaultsKeyLaunchCount = @"ABUserDefaultsKeyLaunchCount";
	NSString * const ABUserDefaultsKeyWWANDownloading = @"ABUserDefaultsKeyWWANDownloading";
	
	@interface ABUserDefaults ()

	+ (void)cacheVersionString;

	@end

	@implementation ABUserDefaults

	+ (void)initialize {
	  [super initialize];
	  [self cacheVersionString];
	}

	+ (NSDictionary*)defaults {
	  return @{ ABUserDefaultsKeyVersionString: @"n/a",
	            ABUserDefaultsKeyLaunchCount: @0,
	            ABUserDefaultsKeyWWANDownloading: @NO
	          };
	}

	+ (int)incrementLaunchCount {
	  int launchCount = ([ABUserDefaults.settings integerForKey:ABUserDefaultsKeyLaunchCount] + 1);
	  [ABUserDefaults.settings setInteger:launchCount forKey:ABUserDefaultsKeyLaunchCount];
	  
	  ABLog(@"[Info] Launch count: %i", launchCount);	  
	  
	  return launchCount;
	}
		
	+ (void)cacheVersionString {
	  NSString* appVersion = NSBundle.mainBundle.infoDictionary[@"CFBundleShortVersionString"];
	  NSString* bundleVersion = NSBundle.mainBundle.infoDictionary[@"CFBundleVersion"];
	  
	  NSString* versionString = [NSString stringWithFormat:@"%@ (%@)", appVersion, bundleVersion];
	  [ABUserDefaults.settings setObject:versionString forKey:ABUserDefaultsKeyVersionString];
	  
	  ABLog(@"[Info] App version: %@", versionString);
	}

	@end