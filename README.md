# grouping
Rxjava+Retrofit+MVP+AOP 类似支付宝微信账单明细列表

## 话不多说先看效果
<br/>![image](https://github.com/tianyah/grouping/blob/master/pic/GIF.gif) </br>





 #                                          怎么用？

			

#### 1. 你的实体类需要实现MultiItemEntity接口


```javascript 
public static class BillInfo implements MultiItemEntity 
```



#### 2. 你需要写一个适配器继承BaseMultiItemQuickAdapter

```javascript

    
  //直接继承BaseMultiItemQuickAdapter单独实现一个适配器的写法

public class BillAdapter extends BaseMultiItemQuickAdapter<BillkEntity.BillInfo, BaseViewHolder> {

    public BillAdapter(List<BillkEntity.BillInfo> data) {
        super(data);
        //这里设置头部界面
        addItemType(BillkEntity.BillInfo.TYPE_HEADER, R.layout.details_head_item);
        //这里设置你的item界面
        addItemType(BillkEntity.BillInfo.TYPE_DATA, R.layout.details_item);
    }

     //根据类型绑定数据
    @Override
    protected void convert(BaseViewHolder helper, BillkEntity.BillInfo item) {
        switch (helper.getItemViewType()) {
               //头部
            case BillkEntity.BillInfo.TYPE_HEADER:
                helper.setText(R.id.item_time, item.billHeaderTime);
                helper.setText(R.id.spending,"支出¥ "+item.spending);
                helper.setText(R.id.income,"收入¥: "+item.income);
                break;
                
                //item数据
            case BillkEntity.BillInfo.TYPE_DATA:


                helper.setText(R.id.trading, item.getBill_name());
                
                //helper.setText(R.id.details_time, StringUtils.getStrTime(item.getCreated_at()));
                helper.setText(R.id.details_time, item.getCreated_at());
                if (item.getMode().equals("0")) {                  
                    ((TextView)helper.getView(R.id.details_item_amount)).
                    setTextColor(mContext.getResources().getColor(R.color.details_text));
                    
                    helper.setText(R.id.details_item_amount, "+" + item.getAmount());
                }
                if (item.getMode().equals("1")) {
                    ((TextView) helper.getView(R.id.details_item_amount)).
                    setTextColor(mContext.getResources().getColor(R.color.black));
                    
                    helper.setText(R.id.details_item_amount, "-" + item.getAmount());
                }
                helper.addOnClickListener(R.id.details_item_r);

                break;

        }
    }
}

```

####  接下来在主界面初始化数据 如果你需要当无网络情况显示网络异常，可以用LoadingView

```javascript
  mPresenter.getBillData(); 
  
LoadingView提供了setEmptyView(View);
                setErrView(View);
                setOnRetryListener(View);


  ```


####  item点击事件

```javascript
        mRecyclerView.addOnItemTouchListener(new OnItemChildClickListener() {

            @Override
            public void onItemClick(BaseQuickAdapter adapter, View view, int position) {

            }

            @Override
            public void onItemChildClick(BaseQuickAdapter adapter, View view, int position) {
                if (view.getId() == R.id.details_item_r) {
                    To.ShowToast(BillActivity.this, "点击了类型二条目的类型为: " +
                    mAdapter.getData(). get(position).getCreated_at());  
                    
                    Log.i("BaseActivity", "onItemChildClick: " + position);
                }
            }

            @Override
            public void onSimpleItemChildClick(BaseQuickAdapter adapter, View view, int i) {

            }
        });


        // 因为添加了1个头部，他是不在clickAdapter.getData这个数据里面的，所以这里要设置数据的偏移值告知ItemDecoration真正的数据索引
        mHeaderItemDecoration.setDataPositionOffset(mAdapter.getHeaderLayoutCount());
    }
   ```
    
    
 ####  无网络点击
 
 ```javascript
     /**
     * 无网络点击回调
     */
    @Override
    public void onRetry() {
        if (!NetworkUtils.isNetworkAvailable(this)) {
            To.ShowToast(this, "请确保网络正常");
            return;
        }
        initData();
}
```
 ####  请求网络数据成功后的回调
 ```javascript
  
    @Override
    public void onSuccess(String bean) {
               
        List<BillkEntity.BillInfo> data = new ArrayList<>();
        
        //这里的数据是你实际项目中的数据类型 ，我这里只是解析本地json测试
        BillkEntity billkEntity = JSONFactory.fromJson(JsonUtils.getJson(this, "rasking.json"), BillkEntity.class);


        data.add(new BillkEntity.BillInfo(BillkEntity.BillInfo.TYPE_HEADER, "2018年1月", "100.01", "28"));
        for (BillkEntity.BillInfo info : billkEntity.January_list) {
            info.setItemType(BillkEntity.BillInfo.TYPE_DATA);
            data.add(info);
        }
        data.add(new BillkEntity.BillInfo(BillkEntity.BillInfo.TYPE_HEADER, "2018年12月", "100.02", "27"));
        for (BillkEntity.BillInfo info : billkEntity.december_list) {
            info.setItemType(BillkEntity.BillInfo.TYPE_DATA);
            data.add(info);
        }
        data.add(new BillkEntity.BillInfo(BillkEntity.BillInfo.TYPE_HEADER, "2018年11月", "100.03", "26"));
        for (BillkEntity.BillInfo info : billkEntity.november_list) {
            info.setItemType(BillkEntity.BillInfo.TYPE_DATA);
            data.add(info);
        }
        data.add(new BillkEntity.BillInfo(BillkEntity.BillInfo.TYPE_HEADER, "2018年10月", "100.04", "25"));
        for (BillkEntity.BillInfo info : billkEntity.october_list) {
            info.setItemType(BillkEntity.BillInfo.TYPE_DATA);
            data.add(info);
        }
        mAdapter = new BillAdapter(data);
        mRecyclerView.setAdapter(mAdapter);

    }
 ```
 #### 无数据
 ```javascript
        mLoadingView.notifyDataChanged(LoadingView.State.empty);
```


####  依赖方式
 ```javascript
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
       }
    }
```

```javascript
  dependencies {
             compile 'com.github.boboyuwu:PinnedHeaderItemDecoration:V1.0.2'
         }
```

#### 如果你需要用到防止重复点击
```javascript 
1,在你的根目录build.gradle下添加
 classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:1.1.0'
 
 然后在你的 app/build.gradle下加上 apply plugin: 'android-aspectjx'
 
 在 dependencies下添加一句  compile 'org.aspectj:aspectjrt:1.8.13'
```
#### aop用法
#### 如果你需要用到防止重复点击，新建一个类
```javascript 
@Aspect
public class SingleClickTest {

    private static final int MIN_CLICK_DELAY_TIME = 800;
    private static int TIME_TAG = R.id.time_click;
    private static final String TAG="SingleClick";
    @Pointcut("execution(@com.example.library.callback.MyClick * *(..))")
    public void methodAnnotated() {
    }

    @Around("methodAnnotated()")
    public void aroundJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
        View view = null;
        for (Object arg : joinPoint.getArgs())
            if (arg instanceof View) view = (View) arg;
        /*
          如果view不为空的遍历添加的注解
         */
        if (view != null) {
            Object tag = view.getTag(TIME_TAG);
            long lastClickTime = ((tag != null) ? (long) tag : 0);
            long currentTime = Calendar.getInstance().getTimeInMillis();
            if (currentTime - lastClickTime > MIN_CLICK_DELAY_TIME) {
                view.setTag(TIME_TAG, currentTime);
                joinPoint.proceed();//执行原方法
            }
        }
    }
}
```
#### 在需要防止重复点击的地方
```javascript

 	
        View.setOnClickListener(new View.OnClickListener() {
	    // 可以是任何点击事件上加一个注解
            @MyClick
            @Override
            public void onClick(View v) {

            }
        });
```
#### 也可以是方法
```javascript
 @MyClick
    public void Test(){
         Log.i(TAG, "Test: ");
    }
```

#### 接下来你就可以愉快的撸代码了
