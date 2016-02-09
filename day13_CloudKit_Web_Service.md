# Day 13 :: CloudKit Web Services

CloudKit JS는 WWDC 15에 소개되었고, 개발자들은 이제 웹 인터페이스를 이용해 유저들은 기존 CloudKit 앱의 Container를 동일하게 사용할 수 있게끔 할 수 있다. CloudKit의 주된 제약 중 하나는 iOS와 OS X 에서만 사용 가능했던(access) 것이다. 이 제약점을 이번에 제거함으로써 더욱 더 많은 앱 개발자들이 CloudKit을 사용하기를 바란다.

이번 포스트에서는, CloudKit JS의 기능을 볼 것이며 샘플 노트 앱을 만들 것이다. 앱을 사용해서 유저들은 중요한 노트들을 클라우드에 저장하게 해 줄 것이다.

## CloudKit JS

CloudKit JS는 아래 브라우저들을 지원한다:

- Safari
- Firefox
- Chrome
- IE
- Edge

흥미롭게도, 노드(nodejs) 역시 지원하는데 이는 여러분의 자체 middle tier 서버로부터의 request를 실행하고 이를 여러분의 API로 포워드 할 수 있게 해 준다.

### CloudKit JS 애플리케이션 만들기

CloudKit JS의 기능을 보여줄 데모앱을 사용하면 공유노트(shared notes)를 CloudKit에 저장할 수 있다.

![The finished web application](images/CloudNotes.png)

자 이제 어떻게 이 앱이 만들어졌는지 하나씩 보자. CloudKit앱을 만들때 첫 걸음은 iCloud developer 대시보드를 오픈하는 것이다(iOS 나 JS나 동일). 이 과정을 통해 여러분들은 앱의 상세사항을 변경할 수 있고, record type 설정, security roles 설정, 데이터입력, 또 기타작업을 할 수 있다. 참고내용은 여기서 찾아보자  [https://icloud.developer.apple.com/dashboard](https://icloud.developer.apple.com/dashboard).

새 앱의 이름은 CloudNotes이다. 우선 모든 세팅은 디폴트로 두자.

앱이 셋업 된 후, 새로운 record type을 지정해야 한다. 이 앱은 제목과 내용을 포함한 간단한 note를 저장한다. 좌측의 'Schema' 아래 'Record Types'옵션을 선택하자. 'Users'record type은 이미 존재해 있을 것이다. 이는 기본적으로 생성된다.

add를 클릭하고 'CloudNote'라는 이름으로 new record type을 생성하자. 이 레코드를 사용해서 데이터를 저장할 예정이다.

![Creating a CloudNote record type](images/cloudNote.png)

그리고 나서 CloudsNote record의 필드에 title, content를 추가하자. (둘다 String 타입)

다음, 데이터를 읽어오고 웹페이지에 표시할만한 record를 생성하자. 좌측메뉴의 "Public Data"섹션 - "Default Zone"을 선택하자. 만들 앱의 모든 데이터는 public 타입이다. 상용 앱에서는 private zone에 데이터를 저장하고 싶을 테지만 이번 tutorial에서는 보안, 권한등은 생략하고 심플하게 제작하겠다.
![Creating a CloudNote instance](images/createNote.png)

add를 클릭하면 title과 content를 넣을 수 있는 옵션이 주어질 것이다. 이후 save를 클릭하면 새 노트 데이터는 CloudKit에 저장될것이다.

이제 CloudKit instance에 데이터를 만들었다. 이제 Javascript를 이용하고 표시해보자.

#### JS 앱 구조

만들 앱은 딱 한 페이지만(index.html) 가질예정이다, 이 파일은 외부 JavaScript파일을 사용하여 CloudKit로 부터 데이터를 요청하거나 저장할 것이다. data가 잘 표시되게 하려고, [Knockout JS](http://knockoutjs.com/)를 사용할 것이다. Knockout은 UI와 CloudKit에서 가져온 데이터 셋 사이의 바인딩을 선언함으로써 작업을 심플하게 해 줄 것이다. 이렇게 함으로써 data model에 변경이 있는 경우 UI가 자동으로 refresh 되게 해줄 것이다. 또한, 별도의 CSS를 작성하지않고, bootstrap으로 부터 스타일을 임포트(import) 할 것이다.

아래에 임포트에 대한 결과 및  CloudKit 임포트 부분이다.

```html
<html>
<head>
  <title>iOS9 Day by Day - CloudKit Web Services Example</title>
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
  <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.3.0/knockout-min.js"></script>
  <script type="text/javascript" src="cloudNotes.js"></script>
  <script src="https://cdn.apple-cloudkit.com/ck/1/cloudkit.js" async></script>
</head>
```

`cloudNotes.js`내용을 보자 그리고 어떻게 우리가 CloudKit로 부터 데이터를 요청할 수 있는지 보자.

데이터를 요청하기에 앞서, 반드시 CloudKit API이 로딩할 시간을 주어야 한다. 따라서 해당 내용을 window eventListener 에 코딩하겠다. 이 코드는 'cloudkitloaded' 이벤트를 모니터링(listens)한다.

```js
window.addEventListener('cloudkitloaded', function() {
```

CloudKit가 일단 로딩된 후 identifier, 환경설정, API Token 등을 이용하여 세팅할 필요가 있다.

```js
CloudKit.configure({
  containers: [{
    containerIdentifier: 'iCloud.com.shinobicontrols.CloudNotes',
    apiToken: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    environment: 'development'
  }]
});
```

API token를 생성하기 위해서는 다시 CloudKit Dashboard로 가자. 'Admin'에서 'API Tokens'를 선택하고 add를 클릭하자. API 토큰의 이름을 정하고 위의 코드(configuration code)상에 복사 붙여넣기 하자.

![API Token Setup](images/apiTokenSetup.png)

`cloudNotes.js`버전은 [Knockout JS](http://knockoutjs.com/) 를 사용하는데, 이는 모델을 HTML과 합치기(bind) 위함이다. 페이지 관리를 위해서 `CloudNotesViewModel`를 생성하였다. 이 모델은 모든 노트에 있는 어레이를 포함하고 있으며, 새로운 노트 저장을 위한 함수, 서버로부터 노트를 가져오거나, 인증상태표시 및 비 인증상태를 표시해준다. (authenticated and unauthenticated state)

뷰 모델이 이러한 함수를 호출하기에 앞서 반드시 CloudKit 인증을 셋업해야 한다.

```js
container.setUpAuth().then(function(userInfo) {
  // Either a sign-in or a sign-out button will be added to the DOM here.
  if(userInfo) {
    self.gotoAuthenticatedState(userInfo);
  } else {
    self.gotoUnauthenticatedState();
  }
  self.fetchRecords(); // Records are public so we can fetch them regardless.
});
```

Promise가 resolved 되면 sign-in 또는 sign-out 버튼이 DOM에 추가될 것이다. (이는 유저의 login 상태에 따름) 따라서 여러분은 div와 함께 페이지상에 "apple-sign-in-button" id를 가져야 한다. 이 `container.setUpAuth()` 함수호출은 div가 적절한 sign in 버튼을 포함하도록 자동으로 수정한다.

#### 레코드 가져오기

fetch records 함수는 'CloudNote'타입으로된 모든 레코드를 쿼리 한다.

```js
self.fetchRecords = function() {
  var query = { recordType: 'CloudNote' };

  // Execute the query.
  return publicDB.performQuery(query).then(function (response) {
    if(response.hasErrors) {
      console.error(response.errors[0]);
      return;
    }
    var records = response.records;
    var numberOfRecords = records.length;
    if (numberOfRecords === 0) {
      console.error('No matching items');
      return;
    }

    self.notes(records);
  });
};
```

위에서 여러분은 어떻게 recordType에 기초한 기본 쿼리가 셋업되는지, 또 퍼블릭(public) 데이터베이스 위에서 실행하는 것을 볼 수 있다. 퍼블릭 데이터베이스에서는 쿼리가 실행될 필요는 없다, 하지만 프라이빗(private) 데이터베이스 에서는 실행될 수 있다. 다만 현재 앱에서 모든 작업은 퍼블릭 이다.

노트가 가져온(fetch) 이후, self.notes로 저장한다(이는 knockout observable임). 무슨 말이냐면 노트 템플릿을 이용해 HTML이 만들어진다는 의미이며, 또 가져와진 노트가 페이지에 나타난다는 뜻이다.

```html
<div data-bind="foreach: notes">
  <div class="panel panel-default">
    <div class="panel-body">
      <h4><span data-bind="text: fields.title.value"></span></h4>
      <p><span data-bind="text: fields.content.value"></span></p>
    </div>
  </div>
</div>
```

위 템플릿은 foreach 바인딩을 이용하여 `notes` 를 반복수행(iterates)한다, 그리고 각 노트의 `fields.title.value`와 `fields.content.value` 값을 패널에 프린트한다.

![A rendered node in the HTML](images/renderedNote.png)

User가 sign in 한 이후, User들은 'Add New Note' 패널을 볼 수 있을 것이다. 따라서 `saveNewNote` 함수는 CloudKit에 새 레코드를 저장할 수 있어야 한다.

```js
if (self.newNoteTitle().length > 0 && self.newNoteContent().length > 0) {
  self.saveButtonEnabled(false);

  var record = {
    recordType: "CloudNote",
    fields: {
      title: {
        value: self.newNoteTitle()
      },
      content: {
        value: self.newNoteContent()
      }
    }
  };
```

함수의 시작 부분을 보면, 레코드가 올바른지 체크하는 기본 밸리데이션을 수행하고, 그리고 현재 폼(form) 에있는 데이터에 근거해서 새로운 레코드를 생성한다.

일단 새로운 레코드가 셋업이 되면, CloudKit에 저장할 준비가 된 것이다.

```js
publicDB.saveRecord(record).then(
  function(response) {
    if (response.hasErrors) {
      console.error(response.errors[0]);
      self.saveButtonEnabled(true);
      return;
    }
    var createdRecord = response.records[0];
    self.notes.push(createdRecord);

    self.newNoteTitle("");
    self.newNoteContent("");
    self.saveButtonEnabled(true);
  }
);
```

`publicDB.saveRecord(record)`라인은 public database에 새로 생성된 레코드를 저장하고 이에 대한 promise를 리턴한다. 생성된 레코드는 기존의 레코드에 속한 어레이로 푸쉬(push)되어 매번 refetch 할 필요성을 없애준다, 그런이후 폼이 초기화되고 save 버튼이 다시 활성화(enable) 된다.

### iOS 앱

CloudKit를 이용하여 어떻게 데이터가 공유 되는지 보여주기 위해서 이 블로그포스트에 iOS 앱 소스 역시 포함되어 있다.

![Compatible iOS App Screenshot](images/iosApp1.png)

셋업을 위해 Xcode 에서 간단히 master detail application을 만들었다.

iCloud와 잘 동작하게 하기 위해서, Xcode 파일 탐색기의 프로젝트를 선택 후 target - capabilities 를 선택하기 바란다. 거기 iCloud를 on으로 선택하자. 스위치를 클릭 하면 Xcode가 개발자센터와 통신을해서 여러분의 앱에 필요한 entitlements를 추가할 것이다.

Configuration 페널은 이제 아래처럼 보일 것이다.

![XCode configuration setup](images/xcodeSetup.png)

여러분들은 이제 CloudKit를 사용할 준비가 되었다. 더이상 자세한 내용을 설명하지는 않겠다. 왜냐면 이미  [comprehensive explanation of how to use CloudKit on iOS in iOS8-day-by-day](https://www.shinobicontrols.com/blog/ios8-day-by-day-day-33-cloudkit) 가 준비되어 있기 때문이다, 다만 우선 이 강좌를 보면 note를 선택할 때 앱이 어떤식으로 작동하는지 알 수 있다. title은 view controller title이 되고, 내용(contents)는 화면 중앙에 표시되어야 한다.

![Compatible iOS App Screenshot](images/iosApp2.png)

## 결론

자 여기까지 CloudKit JS API가 사용하기에 얼마나 심플한지 잘 보셨으리라 생각한다. 저자는 애플이 웹 기반의 CloudKit을 제공해서 기쁘다, 하지만 개인적으로 아직 앱 개발 시 사용하기에는 개선의 여지가 있다고 본다.

이미 시장에는 넘쳐나는 서드파티 클라우드 서비스들이 더 많은 기능과 세련된 문서화로 출시되고 있고, 타 플랫폼을 위한 네이티브 SDK들도 제공된다. 수차례 시도해보았지만 시뮬레이터에서 작동하는 CloudKit 서비스는 아직까지 보지 못했다. 이른 시간 내에 이 사항이 개선되지 않는다면 이는 개발 시 또 다른 장벽이 될 것으로 보인다.

언젠가 CloudKit 를 사용할 기회가 생길 테니 모두 한번 씩 코딩해 볼 것을 권한다. 다만 이른 장래에 타 플랫폼에서 데이터를 엑세스할 필요가 발생할 것이라면 별도 고려해 보시기 바란다.

## 더 읽을거리

앞서 나온 CloudKit JS와 Web Services에 관한 자세한 정보는 WWDC session 710 [CloudKit JS and Web Services](https://developer.apple.com/videos/wwdc/2015/?id=608)를 보기바란다. 이글에서 설명한 프로젝트들을 실행해보고 싶다면 [GitHub](https://github.com/shinobicontrols/iOS9-day-by-day/tree/master/13-CloudKit-Web-Services)에 있으니 잊지 말기 바란다.

만약 질문이나 코멘트가 있다면 우리는 피드백을 듣기를 원한다. [@christhegrant](http://twitter.com/christhegrant)로 트윗을 보내거나, [@shinobicontrols](http://twitter.com/shinobicontrols)를 팔로우해서 iOS9 Day-by-Day 시리즈의 최신 뉴스나 업데이트 소식을 얻을 수 있다!
