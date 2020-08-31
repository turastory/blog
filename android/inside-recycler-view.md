# Inside RecyclerView

안드로이드 앱에서는 리스트를 보여주고 싶을 때, 여러 가지 방법이 있겠지만 일반적으로 RecyclerView를 사용합니다.

이번 글에서는 RecyclerView의 내부 원리에 대해서 알아보겠습니다.



### Concept

RecyclerView는 ViewHolder의 개념을 통해 뷰를 재활용하고 성능을 최적화합니다. 화면에 새로운 아이템을 그릴 때마다 다시 뷰를 만드는 것이 아니라, 만들어놓았던 뷰를 사용해서 화면에 아이템을 보여주게 됩니다.



#### Recycler

scrapped View나 detached View를 관리하는 역할.



##### recycle





##### scrapped

Still attached to RecyclerView, but has been marked for removal or reuse.

-> Scrapped views are for removal or reuse.



##### dirty

if the view is considered as "dirty" the adapter will be asked to **rebind** it.

if not, the view can be quickly reused.

-> RecyclerView re-binds dirty views.



detach되지 않으면 recycle할 수 없음.





#### RecycledViewPool









#### dirty

