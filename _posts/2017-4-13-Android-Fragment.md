---
layout: post
title: Android Fragment
---

## Fragment 정의
> 프래그먼트는 자체 수명 주기를  자체 입력 이벤트를 받으며, 액티비티 실행 중에 추가 및 제거가 가능한 액티비티의 모듈식 섹션이다.

## Fragment 초기화
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    // ... 코드 계속

    FragmentManager fm = getFragmentManager();
    FragmentTransaction fragmentTransaction = fm.beginTransaction();
    fragmentTransaction.add(R.id.fragmentBorC, new FragmentB());
    fragmentTransaction.commit();

    // 코드 계속 ...
}
```
* getFragmentManager() 함수를 사용하여 FragmentManager에 대한 참조를 획득
* FragmentManager의 beginTrasaction() 함수를 호출하여 FragmentTrasaction을 시작한다.
* FragmentTrasaction의 add() 함수를 이용하여 Fragment를 Activity의 ViewGroup(FrameLayout)에 추가한다.
* Fragment와 관련된 모든 작업이 완료되면 FragmentTransaction의 commit() 함수를 호출하여 Fragment와 관련된 작업이 완료되었음을 알려준다.

> Fragment의 Transaction과 관련하여 중요한 내용 중에 `addToBackStack()` 함수가 있다.
> commit()이 실행되기 이전 상태를 "백 스택"에 추가할 수 있는 함수이다. "백 스택"에 들어간 내용은 Activity에 의해 관리되며, "뒤(Back)" 버튼을 누르면 이전 상태에 저장된 Fragment를 다시 가져올 수 있다.

## Fragment 교체
```java
FragmentManager fm = getFragmentManager(); 
FragmentTransaction fragmentTransaction = fm.beginTransaction(); 
fragmentTransaction.replace(R.id.fragmentBorC, fr); 
fragmentTransaction.commit();
```

## Fragment 생성시 데이터 전달
Fragment의 생성자로 데이터를 넘기는 경우 언제 사라질지 모르기 때문에 반드시 setArguments를 통해서 사용해야 한다.
```java
TestFragment frament = new TestFragment();  
Bundle bundle = new Bundle();   
bundle.putInt("id", 1);
frament.setArguments(bundle);
```

저장한 bundle을 가져올 때
```java
Bundle extra = getArguments();
int id = extra.getInt("id");
```

## commitAllowingStateLoss()

commit()은 Activity의 `onSaveInstanceState()` 하기 전에 수행되어야 하며, 이를 어길 시 `java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState` 예외가 발생한다.

예를 들어 화면전환을 했다가 돌아 왔을 때 Fragment 갱신을 해야하는 경우 발생할 수 있다.

> 화면 갱신이 필요한 경우에는 commit()을 쓰지말고 commitAllowingStateLoss()를 호출해서 onSaveInstanceState()와 무관하게 commit를 할 수 있다.
