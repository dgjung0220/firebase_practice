### Firebase Android

1. Firebase 안드로이드 플랫폼 설정
Firebase > Android 앱에 Firebase 추가 > 앱 등록을 마치면 기타 설정 안내 창 확인.
    - Android 패키지 이름 : (실제 안드로이드 스튜디오에서 지정할 패키지명) com.bearpot.firenote
    - 앱 닉네임(선택사항) : firenote

2. 안드로이드 프로젝트 설정
	- Android studio 이용. Firebase 설정에서 했던 명과 동일하게 새 프로젝트를 생성한다.
	- Firebase 안내대로 json 삽입 및 gradle 수정

3. 구글 인증
	- 인증 성공 : 메모 어플리케이션 액티비티로 이동.
	- 인증 실패 : 
		1. 인증 버튼 클릭
		2. 인증창 띄우기
		3. 구글 인증
		4. 인증 액티비티에서 인증 결과 출력
		5. 성공시, 메모 어플리케이션 실행. 실패시, 다시 1번.

	- 인증 액티비티: empty Activity로 생성.

Firebase 인증 모듈 및 Google play service를 앱 수준 gradle 에 의존 주입(add dependencoes) 한다.
```gradle
compile 'com.google.firebase:firebase-auth:11.8.0'
compile 'com.google.android.gms:play-services-auth:11.8.0'
```

GoogleSignin Button
```xml
<com.google.android.gms.common.SignInButton
        android:id="@+id/sign_in_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical|center_horizontal">
    </com.google.android.gms.common.SignInButton>
```
4. 신규 메모 저장
5. 메모 출력
6. 메모 수정 / 삭제
7. 로그 아웃