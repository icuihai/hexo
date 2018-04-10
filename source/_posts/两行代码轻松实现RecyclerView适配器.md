title: "两行代码轻松实现RecyclerView适配器"             #可以随意修改,可以是中文
date: 2016-10-20 15:30:30        #创建的时间,一般不改
categories: blog                 #文章文类
tags: [RecyclerView,Android]                 #文章tag标签,多个标签用逗号隔开
---

> 描述：适用于比较简单的Recyclerview，当然复杂的布局肯定另写一个class比较完美~~

### 如何使用

> 1) In your root build.gradle:
>
> ```Java
> repositories {
>     jcenter()
>     maven { url 'https://jitpack.io' }
> }
> ```
>
> 
>
> 2) In your library/build.gradle add:
>
> ```Java
> compile 'com.github.icuihai:gm-recyclerview:1.0.1'
> ```
>
> 

\###实现过程

- Adapter

  ```
  public abstract class GMRvAdapter<T> extends RecyclerView.Adapter<GMRvViewHolder> {
      protected Context mContext;
      protected int mLayoutId;
      protected List<T> mDatas;
      protected LayoutInflater mInflater;
      private OnItemClickListener mOnItemClickListener;
      private OnItemLongClickListener mOnItemLongClickListener;
  	// 构造函数
  	// mLayoutId :子item布局文件id
  	// List<T>mDatas ：list集合，泛型可以是Module，
      public GMRvAdapter(Context mContext, int mLayoutId, List<T> mDatas) {
          this.mInflater = LayoutInflater.from(mContext);
          this.mContext = mContext;
          this.mLayoutId = mLayoutId;
          this.mDatas = mDatas;
      }

      @Override
      public void onBindViewHolder(GMRvViewHolder holder, final int position) {
  	    // 抽象方法
          convert(holder, mDatas.get(position), position);
          // item点击事件
          if (mOnItemClickListener != null) {
              holder.itemView.setOnClickListener(new View.OnClickListener() {
                  @Override
                  public void onClick(View view) {
                      mOnItemClickListener.onItemClick(view, mDatas.get(position), position);
                  }
              });
          }
          // item长按事件
          if (mOnItemLongClickListener != null) {
              holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                  @Override
                  public boolean onLongClick(View view) {
                      mOnItemLongClickListener.onItemLongClick(view, mDatas.get(position), position);
                      return true;
                  }
              });
          }

      }

      public abstract void convert(GMRvViewHolder easyRvViewHolder, T t, int position);

      @Override
      public GMRvViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
          View view = mInflater.inflate(mLayoutId, parent, false);
          return new GMRvViewHolder(view);
      }
  ```


```java
  @Override
  public int getItemCount() {
      if (mDatas == null) {
          mDatas = new ArrayList<T>();
          return 0;
      }
      return mDatas.size();
  }

  /**
   * 清空数据
   */
  public void clearAll() {
      mDatas.clear();
      notifyDataSetChanged();
  }

  /**
   * 设置RV的item点击监听
   *
   * @param onItemClickListener
   */
  public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
      this.mOnItemClickListener = onItemClickListener;
  }

  public void setOnItemLongClickListener(OnItemLongClickListener onItemLongClickListener) {
      this.mOnItemLongClickListener = onItemLongClickListener;
  }
```


```java
  public interface OnItemClickListener<T> {
      void onItemClick(View view, T data, int position);
  }

  public interface OnItemLongClickListener<T> {
      void onItemLongClick(View view, T data, int position);
  }
```
  }
  ```

- Holder

  ```
  public class GMRvViewHolder extends RecyclerView.ViewHolder {
```java
  private SparseArray<View> mViews;
  private View mConvertView;

  public GMRvViewHolder(View itemView) {
      super(itemView);
      mViews = new SparseArray<>();
      mConvertView = itemView;
  }

  /**
   * 通过viewId获取控件
   */

  public <T extends View> T getView(int viewId) {
      View view = mViews.get(viewId);
      if (view == null) {
          view = mConvertView.findViewById(viewId);
          mViews.put(viewId, view);
      }
      return (T) view;
  }
```
  }
  ```

- 关于我

  > weibo : <https://www.weibo.com/icuihai>
  > gmail : icuihai@gmail.com
  ```

