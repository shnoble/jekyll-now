---
layout: post
title: Android Fragment
---

## Fragment 정의
> 프래그먼트는 자체 수명 주기를  자체 입력 이벤트를 받으며, 액티비티 실행 중에 추가 및 제거가 가능한 액티비티의 모듈식 섹션이다.

Fragment는 Activity를 구성하는 작은 모듈이라고 생각하면  Activity와는 `별도의 라이프 사이클과 이벤트 또한 따로 처리`가  하나의 Activity가 실행되고 있을 때 `유동적으로 추가하고 삭제`가 가능하도록 되어있다.

**Fragment를 사용할 때는 반드시 Activity의 레이아웃에 추가할 필요는 없고 별도의 UI 없이 Activity를 위한 프로세스 처리를 담당할 수도 있다.**

## Fragment Life Cycle
![Fragment Life Cycle](http://cfile27.uf.tistory.com/image/037FB835512F018723A09E)

## 확장 Fragment
기본 Fragment 이외에도 이를 확장하고 있는 보조적인 역할들을 하는 Fragment들이 있다.

* **DialogFragment**: 떠다니는 다이얼로그를 보여주는 Fragment. Fragment는 백스택에 넣어둘 수 있기 때문에 사용자가 다시 Fragment로 복귀하고자 할 때에 Activity에 기본적으로 들어있는 다이얼로그 대신에 사용할수 있는 좋은 대체제이다. 
* **ListFragment**: Adapter를 통해서 List를 보여주는 Fragment로 ListActivity와 비슷하고, list view에서 다룰 수 있는 onListItemClick()과 같은 콜백 함수들도 제공한다.
* **PreferenceFragment**: Preference 객체들을 목록으로 보여주는 PreferenceActivity와 비슷하며, 앱의 Settings를 만들 때에 유용하게 사용할 수 있다.

## Fragment 생성
```java
public static class ExampleFragment extends Fragment { 
    @Override 
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 
        return inflater.inflate(R.layout.example_fragment, container, false); 
    } 
}
```
### inflate()
* inflate를 실행할 레이아웃의 리소스 ID
* inflate한 레이아웃의 부모가 될 ViewGroup. 인자로 받았던 container를 넘겨주는 것은 시스템에서 inflate한 레이아웃을 어디에 그려야할지 정해줘야하는 중요한 단계이다.
* 세번째는 inflate한 레이아웃을 두번째 인자에 추가할 것인지 여부를 묻는 인자로, 시스템에서 자동으로 inflate한 내용을 container에 추가하게 되므로 false를 넘겨줘야 중복되게 추가하는 일이 없다.

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
