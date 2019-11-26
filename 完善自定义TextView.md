# 1. 指定宽高（三种测量模式）

- 如果一个继承自View的自定义View没有复写它的onMeasure()方法 ，会发生什么呢？  View 的 onMeasure() 方法：  

  ```java
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                  getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
      }
  ```

- 接着看getDefaultSize()方法：

  ```java
  public static int getDefaultSize(int size, int measureSpec) {
          int result = size;
          int specMode = MeasureSpec.getMode(measureSpec);
          int specSize = MeasureSpec.getSize(measureSpec);
  
          switch (specMode) {
          case MeasureSpec.UNSPECIFIED:
              result = size;
              break;
          case MeasureSpec.AT_MOST:
          case MeasureSpec.EXACTLY:
              //如果布局中是warp_content那么返回的宽或高都是measureSpec.getSize(widthMeasureSpec)
              //那这个widthiMeasureSpec 在哪里被赋值并传进来的呢？
              //继续看父布局中的onMeasure方法
              result = specSize;
              break;
          }
          return result;
      }
  ```

- 随便先找个Linerlayout.java 进去瞧一瞧：

  ```java
  @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          if (mOrientation == VERTICAL) {
              measureVertical(widthMeasureSpec, heightMeasureSpec);
          } else {
              measureHorizontal(widthMeasureSpec, heightMeasureSpec);
          }
      }
      
      void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
                  ...此处省略部分代码...
                  measureChildBeforeLayout(
                         child, i, widthMeasureSpec, 0, heightMeasureSpec,
                         totalWeight == 0 ? mTotalLength : 0);
      }
    void measureChildBeforeLayout(View child, int childIndex,
              int widthMeasureSpec, int totalWidth, int heightMeasureSpec,
              int totalHeight) {
          measureChildWithMargins(child, widthMeasureSpec, totalWidth,
                  heightMeasureSpec, totalHeight);
      }  
  ```

- 最后调用到ViewGroup.java 中的measureChildWithMargins():

  ```java
  protected void measureChildWithMargins(View child,
              int parentWidthMeasureSpec, int widthUsed,
              int parentHeightMeasureSpec, int heightUsed) {
          final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
          //获取子View的MeasureSpec
          final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                  mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                          + widthUsed, lp.width);
          final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                  mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                          + heightUsed, lp.height);
          //然后传入child.measure 中，这里就是构造子View的MeasureSpec的地方
          child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
      }
  
  ```

- 重点分析getChildMeasureSpec方法

  ```java
  public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
          //spec 父View的MeasureSpec padding 子View左右两边的pading+margin值的和 
          //childDimension 子View布局中的宽或高
          int specMode = MeasureSpec.getMode(spec);
          int specSize = MeasureSpec.getSize(spec);
          //specSize 是父容器的宽或高  padding是子View中设置的具理父容器的值 两个相减得到子View的宽或者高
          int size = Math.max(0, specSize - padding);
  
          int resultSize = 0;
          int resultMode = 0;
  
          switch (specMode) {
          // Parent has imposed an exact size on us
          case MeasureSpec.EXACTLY:
              if (childDimension >= 0) {
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size. So be it.
                  resultSize = size;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size. It can't be
                  // bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
  
          // Parent has imposed a maximum size on us
          case MeasureSpec.AT_MOST:
              if (childDimension >= 0) {
                  // Child wants a specific size... so be it
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size, but our size is not fixed.
                  // Constrain child to not be bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size. It can't be
                  // bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
  
          // Parent asked to see how big we want to be
          case MeasureSpec.UNSPECIFIED:
              if (childDimension >= 0) {
                  // Child wants a specific size... let him have it
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size... find out how big it should
                  // be
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size.... find out how
                  // big it should be
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              }
              break;
          }
          return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
      }
  
  
  ```

  所以从上面代码看出，当我们自定义View中没有重写onMeasure方法的时候，那么它的宽或者高不管设置成match_parent 还是 warp_content 最终都可能是它父View的宽或者高（没有设置任何的pading或者margin情况）
  
- 所以要想我们的自定义View的宽或者高不会是父View那么大，就必须重写OnMeasure方法，通过画笔算出适合自己的宽和高 然后再setMeasuredDimension

  ~~~java
   @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
              super.onMeasure(widthMeasureSpec, heightMeasureSpec);
              //获取宽高的测量模式
              int widthMode = MeasureSpec.getMode(widthMeasureSpec);
              int heightMode = MeasureSpec.getMode(heightMeasureSpec);
  
              int width=MeasureSpec.getSize(widthMeasureSpec);
          int height=MeasureSpec.getSize(heightMeasureSpec);
          //   MeasureSpec.AT_MOST: 在布局中指定的warp_content
          //   MeasureSpec.EXACTLY: 在布局中指定了宽高的具体数值 或者指定了match_parent
          //   MeasureSpec.UNSPECIFIED:一般开发中没有遇到过 Scrollview+listview显示不全的问题
          // 1.确定的值，不需要计算
  
          // 2.给的是warp_content
           if(widthMode==MeasureSpec.AT_MOST){
               Rect rect=new Rect();
               paint.getTextBounds(mText,0,mText.length(),rect);
               width=rect.width();
           }
  
           if(heightMode==MeasureSpec.AT_MOST){
               Rect rect=new Rect();
               paint.getTextBounds(mText,0,mText.length(),rect);
               height=rect.height();
           }
           setMeasuredDimension(width,height);
      }
  
  ~~~

# 2. onDraw 画出文字

![image-20191110094101344](/Users/cailei/Library/Application Support/typora-user-images/image-20191110094101344.png)

要画出文字，就要用到drawText(String text,int x,int y,Paint paint);

方法的参数很简单，`text` 是文字内容，`x`和 `y`是文字的坐标。但需要注意：这个坐标并不是文字的左上角，而是一个与左下角比较接近的位置。大概在这里：

![image-20191113223322397](/Users/cailei/Library/Application Support/typora-user-images/image-20191113223322397.png)

所以我们的重点就求出y的坐标，即基线坐标。

基线坐标如何求呢？首先介绍一下Paint.FontMerticsInt 这个类

```java
  public static class FontMetricsInt {
        //一般top跟bottom对应 ascent和descent对应
        //ascent：文字的最上面的那一条横线距离baseline的距离 一般为负值 
        //descent: 文字的最下边的那条横线距离baseline的距离 一般为正值
        //top: 一般的文字的最上边那条限制横线就够了，但是有些特殊的文字比如A上面带了一个~ 这个~那不就超出了最上面的那条线了吗，所以在ascent的上方留出一定的空间来放置多的空间
       //bottom: 相对应得下方也留出了一定的空间
        public int   top;
        public int   ascent;
        public int   descent;
        public int   bottom;
        public int   leading;

        @Override public String toString() {
            return "FontMetricsInt: top=" + top + " ascent=" + ascent +
                    " descent=" + descent + " bottom=" + bottom +
                    " leading=" + leading;
        }
    }
```

```java
  @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int x = getPaddingLeft();

        //dy 代表的是：高度的一半到 baseLine的距离
        Paint.FontMetricsInt fontMetrics = paint.getFontMetricsInt();
        // top 是一个负值  bottom 是一个正值    top，bttom的值代表是  bottom是baseLine到文字底部的距离（正值）
        // 必须要清楚的，可以自己打印就好
        int dy = (fontMetrics.bottom - fontMetrics.top)/2 - fontMetrics.bottom;
        int baseLine = getHeight()/2 + dy;

        canvas.drawText(mText,x,baseLine,paint);
    }
```



# 3 问题：TextView 继承自 LinerLayout 可以出来效果吗？

答案是出不来效果的。因为默认的viewGroup 不会调用 onDraw()方法。分析源码：

## View.java 

~~~java
public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {
                mOverlay.getOverlayView().dispatchDraw(canvas);

~~~

由元源码可以看出 ：只有当dirtyOpaque 这个值是false 的时候 才会调用draw方法

~~~java
final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
~~~

privateFlags->mPrivateFlags  mPrivateFlags 到底是怎么赋值的？在View的构造函数中最后一行调用computeOpaqueFlages()：

~~~java
  computeOpaqueFlags();
  protected void computeOpaqueFlags() {
        // Opaque if:
        //   - Has a background
        //   - Background is opaque
        //   - Doesn't have scrollbars or scrollbars overlay

        if (mBackground != null && mBackground.getOpacity() == PixelFormat.OPAQUE) {
            mPrivateFlags |= PFLAG_OPAQUE_BACKGROUND;
        } else {
            mPrivateFlags &= ~PFLAG_OPAQUE_BACKGROUND;
        }

        final int flags = mViewFlags;
        if (((flags & SCROLLBARS_VERTICAL) == 0 && (flags & SCROLLBARS_HORIZONTAL) == 0) ||
                (flags & SCROLLBARS_STYLE_MASK) == SCROLLBARS_INSIDE_OVERLAY ||
                (flags & SCROLLBARS_STYLE_MASK) == SCROLLBARS_OUTSIDE_OVERLAY) {
            mPrivateFlags |= PFLAG_OPAQUE_SCROLLBARS;
        } else {
            mPrivateFlags &= ~PFLAG_OPAQUE_SCROLLBARS;
        }
    }
~~~

ViewGroup为啥出不来：

~~~java
private void initViewGroup() {
        // ViewGroup doesn't draw by default
        if (!debugDraw()) {
            setFlags(WILL_NOT_DRAW, DRAW_MASK);
        }
  导致mPrivateFlags 重新赋值      
~~~

解决方法：

1. onDraw 改为dispatchDraw
2. 默认给个透明的背景 setBackground
3. setWillNotDraw()