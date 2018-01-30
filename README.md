# grouping
Rxjava+Retrofit+MVP+AOP 类似支付宝微信账单明细列表

话不多说先看效果
![image](https://github.com/tianyah/grouping/blob/master/pic/GIF.gif) 



怎么用？

你的实体类需要实现MultiItemEntity接口
...public static class BillInfo implements MultiItemEntity ...


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


接下来在主界面初始化数据 如果你需要当无网络情况显示网络异常，可以用LoadingView
LoadingView提供了setEmptyView(View);
                setErrView(View);
                setOnRetryListener(View);

...  mPresenter.getBillData(); ...

事件
... 
    @Override
    protected void initEvent() {
        mLoadingView.setOnRetryListener(this);

        //头部点击事件
        OnHeaderClickAdapter clickAdapter = new OnHeaderClickAdapter() {

            @Override
            public void onHeaderClick(View view, int id, int position) {
                switch (id) {
                    case R.id.details_rl:
                        // case OnItemTouchListener.HEADER_ID:
                        To.ShowToast(BillActivity.this, "头部年月: " + mAdapter.getData().get(position).billHeaderTime);
                        Log.i(TAG, "onHeaderClick: " + "支出: " + mAdapter.getData().get(position).spending);
                        Log.i(TAG, "onHeaderClick: " + "收入: " + mAdapter.getData().get(position).income);

                        break;
                    case R.id.details_image:
                        To.ShowToast(BillActivity.this, "点击了时间图标");
                        break;
                }
            }
        };
        
        PinnedHeaderItemDecoration mHeaderItemDecoration = new PinnedHeaderItemDecoration.
                Builder(BillkEntity.BillInfo.TYPE_HEADER).setDividerId(R.drawable.divider).enableDivider(true)
                .setClickIds(R.id.details_image, R.id.details_rl).disableHeaderClick(false)
                .setHeaderClickListener(clickAdapter)
                .disableHeaderClick(false)
                .setHeaderImageListener(new PinnedHeaderItemDecoration.OnHeaderisShowImageLister() {
                    @Override
                    public void isShow(View view, int position) {
                        Log.i(TAG, "isShow: 当前到达顶部的为 : " + position);
                        //这是当头部到底顶部的时候 这里可以做其它你想做的操作，view可以拿到你头部的任何控件
                        ((ImageView) view.findViewById(R.id.details_image)).setVisibility(View.VISIBLE);

                    }
                })
                .create();
        mRecyclerView.addItemDecoration(mHeaderItemDecoration);

        //item 点击事件
        mRecyclerView.addOnItemTouchListener(new OnItemChildClickListener() {

            @Override
            public void onItemClick(BaseQuickAdapter adapter, View view, int position) {

            }

            @Override
            public void onItemChildClick(BaseQuickAdapter adapter, View view, int position) {
                if (view.getId() == R.id.details_item_r) {
                    To.ShowToast(BillActivity.this, "点击了类型二条目的类型为: " + mAdapter.getData(). get(position).getCreated_at());  
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
    ...
    
    ...   /**
     * 无网络点击回掉
     */
    @Override
    public void onRetry() {
        if (!NetworkUtils.isNetworkAvailable(this)) {
            To.ShowToast(this, "请确保网络正常");
            return;
        }
        initData();
    }
    ...
    
    ... 
    //没有数据
        mLoadingView.notifyDataChanged(LoadingView.State.empty);
        ...
    
    //请求网络数据成功后的回调
    
    ...
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
    ...
    
    
    
    依赖方式
    
    ...
    allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		   }
	    }
            
            dependencies {
             compile 'com.github.boboyuwu:PinnedHeaderItemDecoration:V1.0.2'
         }
            
            ...
            
            接下来你就可以愉快的敲代码了~
