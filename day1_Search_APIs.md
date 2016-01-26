# Day 1: Search APIs

iOS 9 이전에는 앱의 이름으로 찾는 것으로만 spotlight를 사용 할 수 있었다. 새로운 iOS 9의 Search API 발표와 함께, 애플은 개발자가 자신의 App에서 어떤 Content가 Index 되기 원하고 spotlight에서 어떻게 가능한 잘 보여줄지, 또한 그 결과 중 하나를 선택했을 때 어떤 동작이 일어날지를 선택할 수 있게 했다.

## The 3 APIs

### NSUserAcitity

NSUserActivity API는 Handoff를 위해 iOS 8에서 소개되었지만 iOS 9에서 Activity의 검색이 가능하게 되었다. 당신은 이제 이런 activity에 metadata를 줄 수 있고 그것은 spotlight가 activity를 index 할 수 있다는 걸 의미한다. 이 것은 웹에서 browsing 할 때의 history stack과 유사한 역할을 한다. 사용자는 spotlight를 통해 더 빠르게 그의 최근 activity을 열 수 있다.

### Web Markup

Web Markup allows apps that mirror their content on a website to index their content in Spotlight. Users don't need to have the app installed on their device for results to appear in Spotlight. Apple's Indexer will now crawl the web looking for this particular markup in your website. This is then provided to users in both Safari and Spotlight

The fact that results can appear even when your app is **not** installed on a users's device could lead to a lot more exposure to potential users. The deep links from your applications that you expose as public to the Search APIs will be stored in Apple's cloud index. To learn more about Web Markup, take a look at Apples's [Use Web Markup to Make App Content Searchable](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-SW4) documentation.


### CoreSpotlight

CoreSpotlight는 당신의 App 안의 어떤 컨텐츠든 index 할 수 있도록 해주는 새로운 iOS 9의 framework이다. NSUserActivity가 사용자의 history를 저장하는데는 유용하지만 이 API와 함께 당신이 원하는 어떤 data도 index할 수 있다.  이 것은 근본적으로 사용자의 Device의 CoreSpotlight를 통해 low level의 접근을 제공한다.


## Using the Core Spotlight APIs

NSUserActivity와 Web Markup API는 비교적 쉬운데 반해 CoreSpotlight 는 약간 더 복잡하다. 새로운 Core Spotlight API가 어떻게 동작하는지 알아보기 위해, 우리들의 친구 목록을 보여주고 이름을 선택했을 때 사진을 보여주는 간단한 앱을 만들어 보자. Github에서 code를 찾아 그곳에 만들어 둔 것과 함께 따라 갈 수 있다.

![friendApp-576x1024](https://www.shinobicontrols.com/wp-content/uploads/2015/07/friendApp-576x1024.png)

이 app은 간단한 친구의 이름을 보여주는 `FriendTableViewController`와 각 친구에 대해 상세히 보여주는 `FriendViewController`를 포함하는 간단한 storyboard를 갖는다.

![storyboard](https://www.shinobicontrols.com/wp-content/uploads/2015/07/storyboard.png)

우리들의 친구들에 대한 모든 정보는 `Datasource` class에 저장되어 있다. 우리의 친구에 대한 정보를 저장하는 model을 만들고 또한 Core Spotlight index에 친구를 저장하기 위한 logic이 포함 된 곳 이다.
우선, `Datasource` class에서 `Person` object의 목록을 만들고 저장하는 `init()` 함수를 override 한다. 당신은 아마도 database나 어딘가의 server로 부터 받기를 원할 것이지만 데모 목적을 위해 단순히 몇가지 dummy data를 만들것이다.

	override init () {
		let becky = Person()
		becky.name = "Becky"
		becky.id = "1"
		becky.image = UIImage(named: "becky")!
		
		...
		
		people = [becky, ben, jane, pete, ray, tom]
	}

그 data가 `people` array에 저장되면 `Datasource` 를 사용할 준비가 됐다! 
이제 data는 준비가 되어 `FriendTableViewController` 는 table view의 각 셀을 표시하기 위한 요청을 할 때 사용하기 위한 `Datasource`의 instance를 만들 수 있다.

	let datasource = DataSource()
	
`cellForRowAtIndexPath` 함수에서 cell의 내용을 표시하는 건 다음과 같이 간단하다:

	let person = datasource.people[indexPath.row]
	cell?.textLabel?.text = person.name


### Saving the person entries to Core Spotlight

Now the mocked data exists, we can store it in Core Spotlight using the new APIs available in iOS 9. Back in the `Datasource` class, we have defined a function, `savePeopleToIndex`.  The `FriendTableViewController` can call this function when the view has loaded.

In the function, we iterate throuh each person in the `people` array, creating a `CSSearchableItem` for each of them and storing them into a temporary array named `searchableItems`.

	let attributeSet = CSSearchableItemAttributeSet(itemContentType: "image" as String)
	attributeSet.title = person.name
	attributeSet.contentDescription = "This is an entry all about the interesting person called \(person.name)"
	attributeSet.thumbnailData = UIImagePNGRepresentation(person.image)
	
	let item = CSSearchableItem(uniqueIdentifier: person.id, domainIdentifier: "com.ios9daybyday.SearchAPIs.people", attributeSet: attributeSet)
	searchableItems.append(item)
	
The final step is to call `indexSearchableItems` on the default `CSSearchableIdex`.
This actually saves the items into CoreSpotlight so that users can search for them and so they appear in search results.

	CSSearchableIdex.defaultSearchableIndex().indexSearchableItems(searchableItems, completionHandler: { error -> Void in
		if error != nil {
			print(error?.localizedDescription)
		}
	})
	
And that's it! When you run your application, the data will be stored. When you search in spotlight, your friends should appear!

![searchResults1-576x1024](https://www.shinobicontrols.com/wp-content/uploads/2015/07/searchResults1-576x1024.png)


### Responding to User

Now users can see your results in Spotlight, hopefully they will tap on them! But what happens when they do? Well, at the minute, tapping a result will just open the main screen of your app. If you wish to display the friend that the user tapped on, there's a little more work involved. We can specify our app's behaviour when it is opened this way through the `continueUserActivity` `UIApplicationDelegate` method in the app's `AppDelegate`.

Here's  the entire implementation of this method:

	func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
		// Find the ID from the user info
		let friendID = userActivity.userInfo?["kCSSearchableItemActivityIdentifier"] as ! String
		
		// Find the root table view controller and make it show the friend with this ID
		let navigationController = (window?.rootViewController as! UINavigationController)
		navigationController.popToRootViewControllerAnimagted(false)
		let friendTableViewController = navigationController.viewController.first as! FriendTableViewController
		friendTableViewController.showFriend(friendID)
		
		return true
	}
	
As you can see, the information we previously saved into the CoreSpotlight index with the `indexSearchableItems` function is now available to us in the `userActivity.userInfo` dictionary. The only thing we are interested in for the sample is the friend ID, which was stored into the index as the item's kCSSeachableItemActivityIdentifier.

Once we have extracted that information from the `userInfo` dictionary, we can find the application's navigation controller, and pop the root (without animation so it's not noticeable to the user) and then call the `showFriend` function on the `friendTableViewController`. I won't go into detail about how this works, but essentially it finds the friend with the given ID in it's datasource and then pushes a new view controller onto the navigation controller stack. That's all there is to it! Now when the user taps on a friend in spotlight, this is what they will see:

![backToSearch-576xx1024](https://www.shinobicontrols.com/wp-content/uploads/2015/07/backToSearch-576x1024.png)

As you can see, now there is a "Back to Search" option in the top left hand corner of your app. This takes the user directly back to the search screen where they first tapped their friend's name. They can still navigate through the app with the standard back button too.


### Demo Summary

in the demo above, we've seen how easy it is to integrate your application's data with the `CoreSpotlight` index, how powerful it can be when trying to get users to open your app, and how helpful it can be to users looking for  specific content.
We have **not** covered how to remove data from the index, however. This is important and you should anways try to keep the index that your application uses up to date.
For information on how to remove old entries from CoreSpotlight, take a look at the `deleteSearchableItemsWithIdentifiers`, `deleteSearchableItemsWithDomainIdentifiers` and `deleteAllSearchableItemsWithCompletionHander` functions.


## The Importance of Good Citizenship

Although it may seem like a good idea to get as much of your content into Spotlight and Safari as possible, think twice before spamming the search indexes with your content. Being a good citizen in the iOS ecosystem is not only important to keep your customers happy, but Apple will also notice. They have clearly invested a lot into protecting relevance. Engagement ratios are tracked and spammers will be moved to the bottom of search results.


## Further Information

For more information on the new Search APIs, I'd recommend watching WWDC session 709, [Introducing Search APIs](https://developer.apple.com/videos/wwdc/2015/?id=709). You may also be interesting in reading the [NSUserActivity Class Reference](https://developer.apple.com/library/prerelease/ios/documentation/Foundation/Reference/NSUserActivity_Class/) as well as the [documentation for CoreSpotlight](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-SW3).

