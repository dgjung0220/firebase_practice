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
    AuthActivity의 구현
    ```java
    public class AuthActivity extends AppCompatActivity implements GoogleApiClient.OnConnectionFailedListener {

        private SignInButton mSigninBtn;
        private GoogleApiClient mGoogleApiClient;
        private FirebaseAuth mFirebaseAuth;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_auth);

            mSigninBtn = (SignInButton) findViewById(R.id.sign_in_btn);
            mFirebaseAuth = FirebaseAuth.getInstance();

            GoogleSignInOptions googleSignInOptions = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                    .requestIdToken(getString(R.string.default_web_client_id))
                    .requestEmail()
                    .build();

            mGoogleApiClient = new GoogleApiClient.Builder(this)
                    .enableAutoManage(this, this)
                    .addApi(Auth.GOOGLE_SIGN_IN_API, googleSignInOptions)
                    .build();

            mSigninBtn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    Intent intent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
                    startActivityForResult(intent, 100);
                }
            });
        }

        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if (requestCode == 100) {
                // Toast.makeText(AuthActivity.this, "onActivityResult", Toast.LENGTH_LONG).show();

                GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
                GoogleSignInAccount account = result.getSignInAccount();

                if (result.isSuccess()) {
                    firebaseWithGoogle(account);
                } else {
                    Toast.makeText(this, "인증에 실패하였습니다.",Toast.LENGTH_LONG).show();
                }

            }
        }

        @Override
        public void onConnectionFailed(@NonNull ConnectionResult connectionResult) {
            Toast.makeText(this, "인증에 실패하였습니다.",Toast.LENGTH_LONG).show();
        }

        private void firebaseWithGoogle(GoogleSignInAccount account) {
            AuthCredential credential = GoogleAuthProvider.getCredential(account.getIdToken(), null);

            Task<AuthResult> authResultTask = mFirebaseAuth.signInWithCredential(credential);

            authResultTask.addOnSuccessListener(new OnSuccessListener<AuthResult>() {
                @Override
                public void onSuccess(AuthResult authResult) {
                    FirebaseUser firebaseUser = authResult.getUser();
                    Toast.makeText(AuthActivity.this, firebaseUser.getEmail()+"님. 환영합니다.", Toast.LENGTH_LONG).show();
                    startActivity(new Intent(AuthActivity.this, MainActivity.class));
                    finish();
                }
            });
        }
    }
    ```
    ※ GoogleSignIn 기능을 위해서는 SHA-1 인증지문을 firebase 프로젝트 설정에 추가해야 한다.

    MainActivity   
    AuthActivity에서 사용한 인증 정보를 사용하기 위해 아래의 코드를 사용한다.
    ```java
    private FirebaseAuth mFirebaseAuth;
    private FirebaseUser mFirebaseUser;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
    ...

        mFirebaseAuth = FirebaseAuth.getInstance();
        mFirebaseUser = mFirebaseAuth.getCurrentUser();
        if (mFirebaseUser == null) {
            startActivity(new Intent(MainActivity.this, AuthActivity.class));
            finish();
            return;
        }
    ...
    ```
4. 신규 메모 저장
데이터베이스 의존 주입
```gradle
compile 'com.google.firebase:firebase-database:11.8.0'
```
FirebaseDatabase 객체를 얻어온다.
```java
private FirebaseDatabase mFirebaseDatabase;
@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    mFirebaseAuth = FirebaseAuth.getInstance();
    mFirebaseUser = mFirebaseAuth.getCurrentUser();

    mFirebaseDatabase = FirebaseDatabase.getInstance();
	...
```
Memo DTO 를 생성하여, saveMemo method를 작성
```java
private void saveMemo() {
        String text = edContent.getText().toString();

        if (text.isEmpty()) {
            return;
        }

        Memo memo = new Memo();
        memo.setTxt(edContent.getText().toString());
        memo.setCreatedDate(new Date());
        mFirebaseDatabase
                .getReference("memos/"+mFirebaseUser.getUid())
                .push()
                .setValue(memo)
                .addOnSuccessListener(MainActivity.this, new OnSuccessListener<Void>() {
                    @Override
                    public void onSuccess(Void aVoid) {
                        Snackbar.make(edContent, "메모가 저장되었습니다.", Snackbar.LENGTH_LONG).show();
                    }
                });
    }
```
5. 메모 출력
```java
Memo selectedMemo = (Memo)item.getActionView().getTag();
selectedMemoKey = selectedMemo.getKey();
edContent.setText(selectedMemo.getTxt());
```
6. 메모 수정 / 삭제
메모 수정은 기본적으로 저장과 코드가 동일히다. 메모 키만 잘 컨트롤해서 사용하면 된다.
```java
mFirebaseDatabase.getReference("memos/"+mFirebaseUser.getUid()+"/"+selectedMemoKey)
                .setValue(memo)
                .addOnSuccessListener(new OnSuccessListener<Void>() {
                    @Override
                    public void onSuccess(Void aVoid) {
                        Snackbar.make(edContent, "메모가 수정되었습니다.", Snackbar.LENGTH_LONG).show();
                    }
                });
```
실시간으로 수정시 값이 변경되는 것을 확인하기 위해 onChildChanged()를 사용한다.
```java
@Override
public void onChildChanged(DataSnapshot dataSnapshot, String s) {
    Memo memo = dataSnapshot.getValue(Memo.class);
    memo.setKey(dataSnapshot.getKey());
    
    for (int i = 0; i < mNavigationView.getMenu().size(); i++) {
        MenuItem menuItem = mNavigationView.getMenu().getItem(i);
        
        if (memo.getKey().equals(((Memo)menuItem.getActionView().getTag()).getKey())) {
            menuItem.getActionView().setTag(memo);
            menuItem.setTitle(memo.getTitle());
            break;
        }
    }
}
```
삭제의 경우, removeValue() 함수를 이용한다.
```java
if (selectedMemoKey == null) {
            return;
        }

        Snackbar.make(edContent, "메모를 삭제하시겠습니까?", Snackbar.LENGTH_LONG).setAction("삭제", new View.OnClickListener(){

            @Override
            public void onClick(View view) {
                mFirebaseDatabase
                        .getReference("memos/"+mFirebaseUser.getUid()+"/"+selectedMemoKey)
                        .removeValue(new DatabaseReference.CompletionListener() {
                            @Override
                            public void onComplete(DatabaseError databaseError, DatabaseReference databaseReference) {
                                Snackbar.make(edContent, "삭제가 완료되었습니다.", Snackbar.LENGTH_LONG).show();
                            }
                        });
            }
        });
```

7. 로그 아웃
```java
Snackbar.make(edContent, "로그아웃 하시겠습니까?", Snackbar.LENGTH_LONG).setAction("로그아웃", new View.OnClickListener(){
            @Override
            public void onClick(View view) {
                mFirebaseAuth.signOut();
                finish();
                startActivity(new Intent(MainActivity.this, AuthActivity.class));
            }
        });
```
8. 데이터베이스 캐쉬
Firebase는 오프라인시에도 빠르게 데이터베이스 동기화를 위해 캐시를 이용한다. 캐쉬 처리는 한 번만 호출되어야 하므로, static 메서드로 처리한다. (onCreate()에서 호출하면 안 됨)
```java
    private static FirebaseDatabase mFirebaseDatabase;
    static {
        mFirebaseDatabase = FirebaseDatabase.getInstance();
        mFirebaseDatabase.setPersistenceEnabled(true);
    }
```