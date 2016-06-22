# daiily_note
每日一记，常见问题整理


1、activity中有fragment，如果从fragment 执行startActivityForResult方式启动activity，调用谁的OnActivityResult呢？

如果是：getAcitivity().startActivityForResult(intent,code),执行宿主activity的OnActivityResult
        MyFragment.startActivityForResult(intent,code)执行fragment的OnActivityResult
  
  就是谁调用的，就执行谁的方法

2、fragment 嵌套fragment导致子fragment不能执行 OnActivityResult的问题

当我们从一个Activity启动了一个Fragment，然后在这个Fragment中又去实例化了一些子Fragment，

 在子Fragment中去有返回的启动了另外一个Activity，即通过startActivityForResult方式去启动，这时候造成的现象会是，
 子Fragment接收不到OnActivityResult，如果在子Fragment中是以getActivity.startActivityForResult方式启动，
 那么只有Activity会接收到OnActivityResult，如果是以getParentFragment.startActivityForResult方式启动，那么只有父Fragment能接收（
 此时Activity也能接收），但无论如何子Fragment接收不到OnActivityResult。
 
 
 3、android：ListView的局部刷新
 
 private void updateProgressPartly(int progress,int position){
        int firstVisiblePosition = listview.getFirstVisiblePosition();
        int lastVisiblePosition = listview.getLastVisiblePosition();
        if(position>=firstVisiblePosition && position<=lastVisiblePosition){
            View view = listview.getChildAt(position - firstVisiblePosition);
            if(view.getTag() instanceof ViewHolder){
                ViewHolder vh = (ViewHolder)view.getTag();
                vh.pb.setProgress(progress);
            }
        }
    }
    
    
    
 4、动态加载皮肤

说明：加载的方法是通过反射，通过调用AssetManager中的addAssetPath方法，我们可以将一个apk中的资源加载到Resources中，
由于addAssetPath是隐藏api我们无法直接调用，所以只能通过反射，下面是它的声明，通过注释我们可以看出，
传递的路径可以是zip文件也可以是一个资源目录，而apk就是一个zip，所以直接将apk的路径传给它，资源就加载到AssetManager中了，
然后再通过AssetManager来创建一个新的Resources对象，这个对象就是我们可以使用的apk中的资源了，
这样我们的问题就解决了。 
protected void loadResources() {
  try {
    AssetManager assetManager = AssetManager.class.newInstance();
    Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
    addAssetPath.invoke(assetManager, mDexPath);
    mAssetManager = assetManager;
  } catch (Exception e) {
    e.printStackTrace();
  }
  Resources superRes = super.getResources(); //Resources superRes = mContext.getResources();

  mResources = new Resources(mAssetManager, superRes.getDisplayMetrics(),  
        superRes.getConfiguration());  
  mTheme = mResources.newTheme();  
  mTheme.setTo(super.getTheme());  
  }
  
  
  5、RecyclerView瀑布流任性拒绝随机高度
今天，有一个需求，就是做一个瀑布流。第一反应，相当简单，RecyclerView实现瀑布流的方便快捷，早已如雷贯耳，博文遍地了。于是，我踩上了这个惊天大坑了。
我还以为RecyclerView 会为每个item 动态设置高度，然而，呵呵。想得美。

思路
因为瀑布流都会一般都会有图片展示，我举得根据每一张图片的不同的宽高比例，去实现瀑布流。
1.
拿到每一个item里面的ImageView的宽度 计算方式：
屏幕宽度/列数 - item的左边距 - item 的右边距 （如果有内边距则再减掉左右两边的Padding）

mItemWidth = AppUtil.getDisplayMetrics(context).widthPixels / 2 - AppUtil.dip2px(10) - AppUtil.dip2px(5);

在Adapter 的onBindViewHolder()方法中，获取ImageView的bitmap，并计算按比率缩小后的bitmap的高度，有了imageview的宽高，后面就so easy了

//填充onCreateViewHolder方法返回的holder中的控件
@Overridepublic void onBindViewHolder(final DetailsViewHolder holder, final int position) {
    ItemDetails.Details details = mLists.get(position);
    holder.tvLikeCount.setText(String.valueOf(details.getLikeCount()));
    ImageLoader.getInstance().load(holder.ivPic, details.getPorposeUrl());
    ImageLoader.getInstance().load(details.getPorposeUrl(), new ImageLoader.OnLoadByteListener() {
        @Override
        public void onLoad(Bitmap resource) {
            int height = ImageUtil.getHeight(resource, mItemWidth);
            resource.recycle();//算完高度，回收bitmap，释放内存
            RelativeLayout.LayoutParams params = new RelativeLayout.LayoutParams(mItemWidth, height);
            holder.ivPic.setLayoutParams(params);
        }
    });
}

根据宽度计算等比例的高度的方法

public static  int getHeight(Bitmap btm,int newWidth){
    float btmW=btm.getWidth();
    float btmH=btm.getHeight();
    float scale=Math.abs((btmW-newWidth))/btmW;
    return Math.abs((int)(btmH-btmH * scale));
}


6、 判断ScrollView 是否滚动到底部或顶部
1）是否滚动到顶部
[java] view plain copy print?
if(scroll.getScrollY() == 0){  
    // 到顶部了   
}  

2）是否滚动到底部
//childView是scrollview里包含的Linearlayout容器  
View childView = scrollView.getChildAt0);  
if(mLastY == (childView.getHeight()-scrollView.getHeight())){  
    //滑动到底部  
}  
