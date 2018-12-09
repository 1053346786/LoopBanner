# LoopBanner

## 功能介绍

- 支持自动无限轮播，无需手动开启和关闭，无需担心内存泄漏
- 支持页面内容完全自定义
- 支持自定义页面点击事件
- 支持自定义指示器样式

基本上该有的都有，并且使用简单，只要你会用ListView

## 使用介绍

### 1.添加Gradle依赖

在项目的``build.gradle``文件中添加``jitpack``仓库地址

```groovy
allprojects {
    repositories {
        ...
        //添加下面的仓库地址
        maven { url "https://jitpack.io" }
    }
}
```

在``app``的``build.gradle``文件中添加``dependencies``：

```groovy
dependencies {
    ...
    implementation 'com.github.wenjiangit:LoopBanner:1.0.0'
}
```

版本更新：[点击查看最新版本号](https://github.com/wenjiangit/LoopBanner/releases)

### 2.在xml布局资源中引用

```xml
 <com.wenjian.loopbanner.LoopBanner
            android:id="@+id/lb1"
            android:layout_width="match_parent"
            android:layout_height="130dp"
            app:lb_indicatorStyle="normal"
            app:lb_pageMargin="4dp"
            app:lb_lrMargin="16dp"
            app:lb_interval="4000" />
```

有关的属性定义后面再详细介绍

### 3.绑定数据

绑定数据有两种方式

1. 直接设置图片资源，并添加点击事件

```kotlin
    //设置图片资源并添加page点击事件
        lb1.setImages(DataCenter.loadImages()) { _, position ->
            Toast.makeText(this, "position=$position", Toast.LENGTH_SHORT).show()
        }
```

当然如果你不需要点击事件，那就更简单了，如下：

```kotlin
//直接绑定图片资源
lb1.setImages(DataCenter.loadImages())
```

2. 通过``setAdapter``绑定数据

```kotlin
//直接设置adapter,默认的itemView是ImageView
lb2.adapter = object : LoopAdapter<String>(DataCenter.loadImages()) {
    override fun onBindView(holder: ViewHolder, data: String, position: Int) {
        val itemView = holder.itemView as ImageView
        Glide.with(holder.context).load(data).into(itemView)
        itemView.setOnClickListener {
            Toast.makeText(this@MainActivity, "position=$position", Toast.LENGTH_SHORT).show()
        }
    }
}
```

``Adapter``的构建也很简单，只需要继承``LoopAdapter``，传入数据列表，并重写``onBindView``方法即可,在``onBindView``中可以设置点击事件，并绑定控件，与``RecylerView``的``onBindView``思想是一样的。

需要注意的是``LoopAdapter``接受两个参数，如下

```java
public LoopAdapter(List<T> data, int layoutId) {
    mData = data;
    mLayoutId = layoutId;
}
```

``layoutId``如果不传，默认的``holder.itemView``是``ImageView``,否则就是加载对应的布局而生成的``view``。如下：

```kotlin
class MyAdapter(data: List<BannerEntity>) : LoopAdapter<BannerEntity>(data, R.layout.lay_banner_item) {
    override fun onBindView(holder: ViewHolder, data: BannerEntity, position: Int) {
        val image = holder.getView<ImageView>(R.id.iv_image)
        Glide.with(holder.context).load(data.url).into(image)
        holder.setText(R.id.tv_title,data.title)

        holder.itemView.setOnClickListener {
            Toast.makeText(holder.context, "position=$position", Toast.LENGTH_SHORT).show()
        }
    }
}
//设置Adapter
lb3.adapter = MyAdapter(DataCenter.loadEntities())
```

``ViewHolder``提供getView方法获取对应的控件，与``holder.itemView.findViewById()``的效果是一样的，不过``getView``做了缓存，可以减少每次``findViewById``的时间。

### 4.设置指示器风格（可选）

1. 属性设置

````xml
//普通风格，小圆点(默认样式)
app:lb_indicatorStyle="normal"
//京东app首页广告图指示器风格，挺独特的
app:lb_indicatorStyle="jd"
//小药丸风格，选中的小圆点变成小药丸
app:lb_indicatorStyle="pill"
````

2. 代码设置

```kotlin
lb1.setIndicatorStyle(LoopBanner.Style.NORMAL)
lb1.setIndicatorStyle(LoopBanner.Style.JD)
lb1.setIndicatorStyle(LoopBanner.Style.PILL)
```

到此为止，经过以上的设置已经可以实现市场上大部分的效果了。

## 补充说明

这部分适用于对于UI效果或者指示器风格有更高要求的人

### 1.自定义属性说明

```xml
<!--page之间的间距-->
<attr name="lb_pageMargin" format="dimension|reference" />
<!--是否无限轮播-->
<attr name="lb_canLoop" format="boolean" />
<!--中间page离父布局的左右间距-->
<attr name="lb_lrMargin" format="dimension|reference" />
<!--page的上间距-->
<attr name="lb_topMargin" format="dimension|reference" />
<!--page的下间距-->
<attr name="lb_bottomMargin" format="dimension|reference" />
<!--轮播周期-->
<attr name="lb_interval" format="integer" />
<!--离屏缓存个数-->
<attr name="lb_offsetPageLimit" format="integer" />
<!--page的上下左右间距，如果同时设置单独方向的间距，会被覆盖掉-->
<attr name="lb_margin" format="dimension|reference" />

<!--是否显示指示器-->
<attr name="lb_showIndicator" format="boolean" />
<!--指示器位置-->
<attr name="lb_indicatorGravity">
    <flag name="bottomLeft" value="1" />
    <flag name="bottomRight" value="2" />
    <flag name="bottomCenter" value="3" />
</attr>
<!--单个指示器大小-->
<attr name="lb_indicatorSize" format="dimension"/>
<!--单个指示器之间的间距-->
<attr name="lb_indicatorMargin" format="dimension|reference"/>
<!--选中的指示器的drawable资源-->
<attr name="lb_indicatorSelect" format="reference"/>
<!--未选中的指示器的drawable资源-->
<attr name="lb_indicatorUnSelect" format="reference"/>
<!--指示器容器垂直方向的间距-->
<attr name="lb_indicatorParentMarginV" format="dimension|reference"/>
<!--指示器容器水平方向的间距-->
<attr name="lb_indicatorParentMarginH" format="dimension|reference"/>
<!--指示器风格样式-->
<attr name="lb_indicatorStyle">
    <enum name="normal" value="0"/>
    <enum name="jd" value="1"/>
    <enum name="pill" value="2"/>
</attr>
```

每个属性都有详细注释，就不单独解释了。

### 2.指示器位置和样式

位置设置:

``lb_indicatorGravity``：定义指示器处于左下，中下，右下

``lb_indicatorParentMarginV``：定义指示器垂直向下的间距

``lb_indicatorParentMarginH``：定义指示器水平方向的间距，靠左还是靠右由``lb_indicatorGravity``决定

这3个属性配合使用基本上可以定位到父容器内的任何位置。

样式设置：

``lb_indicatorSelect``：选中的drawable资源

``lb_indicatorUnSelect``：未选中的drawable资源

这两个配合使用可以实现80%的效果，另外的20%需要通过以下方式实现

实现``IndicatorAdapter``接口,每个方法的作用可以看注释

```java
public interface IndicatorAdapter {

    /**
     * 添加子indicator
     *
     * @param container 父布局
     * @param drawable  配置的Drawable
     * @param size      配置的指示器大小
     * @param margin    配置的指示器margin值
     */
    void addIndicator(LinearLayout container, Drawable drawable, int size, int margin);

    /**
     * 应用选中效果
     *
     * @param prev    上一个
     * @param current 当前
     * @param reverse 是否逆向滑动
     */
    void applySelectState(View prev, View current, boolean reverse);

    /**
     * 应用为选中效果
     *
     * @param indicator 指示器
     */
    void applyUnSelectState(View indicator);


    /**
     * 是否需要对某个位置进行特殊处理
     *
     * @param container 指示器容器
     * @param position  第一个或最后一个
     * @return 返回true代表处理好了
     */
    boolean handleSpecial(LinearLayout container, int position);


}
```

以下是默认的实现``SelectDrawableAdapter``，大家可以参考下

```java
public final class SelectDrawableAdapter implements IndicatorAdapter {

    private LinearLayout.LayoutParams mLayoutParams;

    @Override
    public void addIndicator(LinearLayout container, Drawable drawable, int size, int margin) {
        LinearLayout.LayoutParams layoutParams = generateLayoutParams(drawable, size, margin);
        ImageView image = new ImageView(container.getContext());
        image.setImageDrawable(drawable);
        container.addView(image, layoutParams);
    }

    @Override
    public void applySelectState(View prev, View current, boolean reverse) {
        current.setSelected(true);
        current.requestLayout();
    }

    private LinearLayout.LayoutParams generateLayoutParams(Drawable drawable, int size, int margin) {
        if (mLayoutParams == null) {
            final int minimumWidth = drawable.getMinimumWidth();
            final int minimumHeight = drawable.getMinimumHeight();
            LinearLayout.LayoutParams layoutParams;
            if (minimumWidth == 0 || minimumHeight == 0) {
                layoutParams = new LinearLayout.LayoutParams(size, size);
            } else {
                layoutParams = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT);
            }
            layoutParams.leftMargin = margin;
            mLayoutParams = layoutParams;
        }
        return mLayoutParams;
    }

    @Override
    public void applyUnSelectState(View indicator) {
        indicator.setSelected(false);
        indicator.requestLayout();
    }

    @Override
    public boolean handleSpecial(LinearLayout container, int position) {
        return false;
    }
```

除此之外，还有一个``JDIndicatorAdapter``,这里就不贴代码了，感兴趣的可以自己看看。

### 3.页面切换效果

观察了市面上大多数知名app后，发现并没有使用多么炫酷的切换效果，都是一般的平滑过度，所以这里也没有内置切换效果，不过开放了接口，有需求的同学可以自己设置。

```java
public void setPageTransformer(ViewPager.PageTransformer pageTransformer)
```

### 4.图片圆角和硬件加速

如果需要图片或者``CardView``的圆角效果，就不要设置``lb_lrMargin``,否则你设置任何圆角效果都是毫无意义的，这关系到硬件加速，具体原因还不清楚。

## 感谢

[**BaseRecyclerViewAdapterHelper**](https://github.com/CymChad/BaseRecyclerViewAdapterHelper)

[**glide**](https://github.com/bumptech/glide)

[**wanandroid**](http://www.wanandroid.com/)

## 联系我

>qq:824636515
>
>邮箱：824636515@qq.com


