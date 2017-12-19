### Coding Firebase
- [데모](https://memowebapp-aafcc.firebaseapp.com)
- [Project](../memoWeb)
- [index.html](../memoWeb/public/index.html) - materialize UI template 사용
	* memoWeb 구현될 기능 설명
		* 메모 입력 및 신규 메모 생성
		* 메모 리스트 생성
		* 메모 리스트를 통한 메모 열람

1. 스크립트 설정
[firebase](firebase.google.com)> Go to Console > memoWebApp > 웹 앱에 Firebase 추가 > 스니펫 복사하여 index.html 의 하단부에 붙여 넣는다.
```js
<script src="https://www.gstatic.com/firebasejs/4.8.0/firebase.js"></script>
<script>
  // Initialize Firebase
  var config = {
    apiKey: "",
    authDomain: "",
    databaseURL: "",
    projectId: "",
    storageBucket: "",
    messagingSenderId: ""
  };
  firebase.initializeApp(config);
</script>
```

2. 인증기능을 이용한 구글창 호출
인증이 성공하면 추후 구현될 메모리스트를 출력하고, 실패시 인증 팝업을 띄운다.
```js
auth = firebase.auth();
var authProvider = new firebase.auth.GoogleAuthProvider();
auth.onAuthStateChanged(function(user) {
	if(user) {
    	console.log("success");
        console.log(user)
        // 메모 리스트 출력
    } else {
    	auth.signInWithPopup(authProvider);
    }
})
```

3. 구글 인증 성공 - 메모 리스트 출력
[database](https://console.firebase.google.com/u/0/project/memowebapp-aafcc/database/memowebapp-aafcc/data)의 이용. firebase database tab에서 'memos' 라는 DB TABLE을 만들어 진행한다.
```js
var database;
database = firebase.database();
```
```js
// 메모 리스트 출력 함수
function get_memo_list() {
	console.log(userInfo.uid);
    var memoRef = database.ref('memos/' + userInfo.uid);
          
    memoRef.on('child_added', function(data) {
    	console.log(data.val());
    });
}
```

4. 메모 저장(.push)
firebase의 push 함수를 이용하여 데이터를 저장한다.
```js
function save_data() {
	var memoRef = database.ref('memos/' + userInfo.uid);
	var txt = $(".textarea").val();
    
    // txt 가 비어있으면, 저장하지 않는다.
    if(txt = '') {return;}
    
	//push
	memoRef.push({
		txt : txt,
		createDate : new Date().getTime() 
	})
}
```
5. 메모 한 건 출력(.once)
데이터 하나를 불러올 때는 key값을 이용해 db에 접근한 후, once 함수를 이용한다. 이 때 once 함수는 promise를 리턴하게 되고, then() 메소드를 이용하여 아래와 같이 처리한다.
```js
var memoRef = database.ref('memos/' + userInfo.uid+'/'+key)
        .once('value').then(function(snapshot){
        	$(".textarea").val(snapshot.val().txt);
        });
```
6. 메모 수정(.update)
```js
memoRef = database.ref('memos/' + userInfo.uid + '/' + selectedKey);
memoRef.update({
	txt : txt,
	createDate : new Date().getTime(),
	updateDate : new Date().getTime()
});
```
리스트에서 수정된 데이터 실시간 반영(child_changed). child_added나 child_changed 등 데이터의 추가, 변경시 실시간으로 메소드가 동작할 수 있는 것은 실시간 DB를 지원하는 firebase의 중요한 특징이다.
```js
memoRef.on('child_changed', function(data){
              console.log(data.key);
              console.log(data.val());

              var key = data.key;
              var txt = data.val().txt;
              var title = txt.substr(0, txt.indexOf('\n'));

              $("#"+key+ ">.title").text(title);
              $("#"+key+ ">.txt").text(txt);
          });
```
7. 메모 삭제(.remove)
```js
if (!confirm('삭제하시겠습니까?')) {
	return;
}
var memoRef = database.ref('memos/' + userInfo.uid+'/'+key);
memoRef.remove();			// firebase data remove

            $("#"+key).remove();
```

8. 서버 호스팅
```
$ firebase deploy
```
firebase cli 에서 firebase deploy 명령어를 이용해서 배포한다. 배포 진행이 완료되면, Deploy complete! 명령과 함께 아래와 같이 Hosting URL이 출력된다. (※ Firebase는 http 지원하지 않는다. 배포 전에 소스코드에 http로 접근하는 CDN 이 있다면 수정할 것.)
```
=== Deploying to 'memowebapp-aafcc'...
i  deploying database, hosting
i  database: checking rules syntax...
✔  database: rules syntax for database memowebapp-aafcc is valid
i  hosting: preparing public directory for upload...
✔  hosting: 1 files uploaded successfully
i  database: releasing rules...
✔  database: rules for database memowebapp-aafcc released successfully
✔  Deploy complete!
Project Console: https://console.firebase.google.com/project/memowebapp-aafcc/overview
Hosting URL: https://memowebapp-aafcc.firebaseapp.com
```