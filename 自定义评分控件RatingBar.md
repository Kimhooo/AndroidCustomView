# 自定义评分View效果分析

1.  两张星星图片
2. 星星数量

# 自定义属性

~~~java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="RatingBar">
        <attr name="starNormal" format="reference"/>
        <attr name="starFocus" format="reference"/>
        <attr name="starNum" format="integer"/>
    </declare-styleable>
</resources>

~~~



# 获取自定义属性

~~~java
  TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.RatingBar);

        int starNormalId = typedArray.getResourceId(R.styleable.RatingBar_starNormal, 0);

        if (starNormalId == 0) {
            throw new RuntimeException("请设置属性 starNormal");
        }
        mStarNormalBitmap = BitmapFactory.decodeResource(getResources(), starNormalId);

        int starFocusId = typedArray.getResourceId(R.styleable.RatingBar_starFocus, 0);

        if (starFocusId == 0) {
            throw new RuntimeException("请设置属性 starNormal");
        }

        mStarFocusBitmap = BitmapFactory.decodeResource(getResources(), starFocusId);

        mStarNum = typedArray.getInt(R.styleable.RatingBar_starNum, mStarNum);
~~~



# 重写onMeasure()方法

~~~java
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //指定控件宽高 控件的高即是星星的高

        int starHeight = mStarFocusBitmap.getHeight();
        int starWidth = mStarFocusBitmap.getWidth();

        int height = starHeight + getPaddingBottom() + getPaddingTop();
        int width = 0;
        for (int i = 0; i < mStarNum; i++) {
            width = starWidth + width + mSpace;
        }


        setMeasuredDimension(width + getPaddingLeft() + getPaddingRight(), height);


    }
~~~



# 画出对应数量的星星

```java
for (int i = 0; i < mStarNum; i++) {
    canvas.drawBitmap(mStarNormalBitmap, mStarFocusBitmap.getWidth() * i + mSpace * i, 0, null);
}
```



# 触摸事件处理

~~~java
 public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
//                int moveX= (int) event.getX();
//                mNeedDraw=getNeedDrawNum(moveX);
//                if(mNeedDraw>0){
//                    invalidate();
//                }
            case MotionEvent.ACTION_MOVE:
                int moveX= (int) event.getX();
                mNeedDraw=getNeedDrawNum(moveX);
                if(mNeedDraw>0){
                    invalidate();
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
    //这里一定要reture true 如果reture super.onTouchEvent 就是false false 表示不消耗这个事件，就不会继续调用   Move 方法了 
        return true;
    }
~~~

