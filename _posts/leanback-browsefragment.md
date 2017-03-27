---
title: leanback_browsefragment
date: 2017-03-08 21:18:42
tags:
---
#### Leanback
在Android TV 中 Leanback 这个词会经常看见,比如 LeanbackLauncher, Leanback support library. 然而并没有找到 Google 对 Leanback 这个东西的明确定义,姑且就认为是 Android TV 特有的一种交互方式吧.

#### v17 Leanback support library
```com.android.support:leanback-v17:24.2.0```

这个库提供了一些 Leanback 风格的控件和主题,方便快速开发一个标准的Android TV 程序.

{% asset_img atv-leanback-all.png %}

就是图上面这些,基本把内容套进去就是一个像样的Android TV应用了.

#### 声明
Android TV应用需要在`AndroidManifest.xml` 声明使用leanback,来表示这是一个Android TV应用.

```<uses-feature android:name="android.software.leanback" android:required="false" />```

入口activity 也有些不大一样,`inter-filter`的category 需要申明为`LEANBACK_LAUNCHER`.

```
<intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
</intent-filter>
```

theme 最好继承自 Lbeanback supoort library 中的主题,而不是 appcompat.

```<style name="AppTheme" parent="Theme.Leanback">```

TV的launcher不再显示正方形的图标和应用名,而是用一块长方形的banner来表示一个应用.

```android:banner="@drawable/banner"```

所以一个完整的Android TV应用应该在`application`或者`activity`字段声明一个banner属性.
否则Android TV不知道拿什么来显示的应用.

#### BrowseFragment
接下来看重点.`BrowseFragment`应该是Android TV最常见的一种展现形式.
```
public class LeanActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_lean);
    }
}
```
先来创建一个activity,注意这里直接集成Activity,而不是`AppCompatActivity`.另外,不要忘了改`intent-filter`和加banner.
再来创建一个fragment,继承`BrowseFragment`

```
public class LeanFragment extends BrowseFragment {

}
```
然后把`LeanFragment` 添加到`LeanFragment`的根布局.

使用`BrowseFragment`不需要自定义任何布局,有点类似`ListActivity`的意思.此时,我们的应用已经能够跑起来了.

{% asset_img awesome_banner.png %}

在Launcher上看到我们的应用了.

启动之后是这样的:


开始往里面填充数据.
{% asset_img full.png %}

一个完整的界面是这样,有若干可以横向滚动的行,每行有一个标题和若干个子项.

一行内容使用一个 ListRow 来描述

`new ListRow(new HeaderItem("catetory0"), new ListRowPresenter())`

一个 row 需要一个`HeaderItem`来表示分类,以及一个 adapter 来获取这个 row 对应的数据条目.

`BrowseFragment`内置一个 RecyclerView,那么我们还需要一个 adapter 来把 row 数据展示在 recyclerView 里.
```
mCategoryRowAdapter = new ArrayObjectAdapter(new ListRowPresenter());
```
Leanback 为我们包装好了Adpater 实现,不需要自己实现一个 Adapter.这里我们使用`ObjectAdapter`的子类`ArrayObjectAdapter`,对应静态数据,也有`CursorObjectAdapter`对用游标数据源.

`ObjectAdapter`的初始化需要一个`Presenter`,作用是把一个数据对象映射到一个 ViewHolder 上去.因为`BrowseFragment`的数据对象和`ViewHolder` 都是内置的,所以我们直接使用内置的`ListRowPresenter`实现就好了.

```
mCategoryRowAdapter = new ArrayObjectAdapter(new ListRowPresenter());
ListRow row0 = new ListRow(new HeaderItem("catetory0"), new ArrayObjectAdapter());
mCategoryRowAdapter.add(row0);
setAdapter(mCategoryRowAdapter);
```

跑起来大概长这样,只有一个标题,后面没有数据.

{% asset_img row0.png %}

现在来往 row 里面添加数据.我们只需要构造 `ListRow`的第二个参数 `ArrayObjectAdapter`即可.

上面说到,显示数据需要实现一个`Presenter`,do it.

主要工作就3个.

`onCreateViewHolder`中返回一个带 View 的 ViewHolder,这里可以先不管大小和内容.

`onBindViewHolder`中设置 View 的内容,这里会传递数据对象和 ViewHolder 对象.强制转换和赋值即可.

`onUnbindViewHolder`中清空 View 的内容.

```
class CardPresenter extends Presenter {

    private Drawable mDefaultCardImage;

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent) {
        mDefaultCardImage = parent.getResources().getDrawable(R.drawable.movie, null);
        ImageCardView cardView = new ImageCardView(parent.getContext());
        return new ViewHolder(cardView);
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, Object item) {
        Video video = (Video) item;

        ImageCardView cardView = (ImageCardView) viewHolder.view;
        cardView.setTitleText(video.title);
        cardView.setContentText(video.studio);

        if (video.cardImageUrl != null) {
            Resources res = cardView.getResources();
            int width = res.getDimensionPixelSize(R.dimen.card_width);
            int height = res.getDimensionPixelSize(R.dimen.card_height);
            cardView.setMainImageDimensions(width, height);

            Glide.with(cardView.getContext())
                    .load(video.cardImageUrl)
                    .error(mDefaultCardImage)
                    .into(cardView.getMainImageView());
        }
    }

    @Override
    public void onUnbindViewHolder(ViewHolder viewHolder) {
        ImageCardView cardView = (ImageCardView) viewHolder.view;
        cardView.setBadgeImage(null);
        cardView.setMainImage(null);
    }
}
```

构造ListRow 的用上自己的`Presenter`:

```java
ArrayObjectAdapter rowAdapter = new ArrayObjectAdapter(new CardPresenter());
//填充数据
int items = 10;
while ((items--) > 0) {
    Video video = new Video("title","studio","https://img3.doubanio.com/view/photo/thumb/public/p2388501883.jpg");
    rowAdapter.add(video);
}
ListRow row0 = new ListRow(new HeaderItem("catetory0"), rowAdapter);
```
一个完整`BrowseFragment`就展现出来了.
{% asset_img row0_data.png %}

#### 总结
Google 说 Leanback 这套玩意就是江湖上的 MVP 模式,可是这 Presenter 跟别人家的 Presenter 长的不大一样啊.

其实 G家的Presenter 可以当做一个映射起

还是一些点没涉及到,比如更新背景:

```
mBackgroundManager = BackgroundManager.getInstance(getActivity());
mBackgroundManager.attach(getActivity().getWindow());
mBackgroundManager.setBitmap(resource);
```

几个 lisitener:

```
setOnSearchClickedListener
setOnItemViewClickedListener
setOnItemViewSelectedListener
```

