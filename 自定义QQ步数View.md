# QQ计步效果分析

![image-20200206184930553](https://tva1.sinaimg.cn/large/006tNbRwly1gbmvj5hogcj30le0mmwka.jpg)

1. 先画出外面的蓝色的外圆 
2. 画出里面的红色的内圆
3. 画出中间的文字

# 自定义View分析的常用步骤

1. 分析效果
2. 确定自定义属性，编写attr.xml 文件
3. 在布局中使用
4. 在自定义View中获取自定义属性
5. 开始具体逻辑画View

# 自定义属性

~~~java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="QQStepView">
        <attr name="outColor" format="color" />
        <attr name="innerColor" format="color" />
        <attr name="stepTextColor" format="color" />
        <attr name="stepTextSize" format="dimension" />
        //圆环宽度 外圆减去内圆
        <attr name="borderWidth" format="dimension"/>
        //内圆半径
        <attr name="radius" format="dimension"/>
    </declare-styleable>
</resources>

~~~



# 获取自定义属性

```java
        TypedArray array=context.obtainStyledAttributes(attrs,R.styleable.QQStepView);
        mInnerColor=array.getColor(R.styleable.QQStepView_innerColor,Color.RED);
        mOutColor=array.getColor(R.styleable.QQStepView_outColor,Color.BLUE);
        mBorderWidth= (int)array.getDimension(R.styleable.QQStepView_borderWidth,mBorderWidth)       mStepTextSize=array.getDimensionPixelSize(R.styleable.QQStepView_stepTextSize,mStepTextSize);
        mStepTextColor=array.getColor(R.styleable.QQStepView_stepTextColor,mStepTextColor);


        initPaints();
        array.recycle();
```

# 画外圆弧

```java
 //startAngle 开始角度 x轴正向是0度 sweepAngle 顺时针扫过的角度
        RectF rectF=new RectF(0,0,getWidth(),getHeight());
        canvas.drawArc(rectF,135,270,false,mOutPaint);
```

现象：![image-20200211102829406](https://tva1.sinaimg.cn/large/0082zybply1gbs95cel30j30b80asa9x.jpg)

1. 可以发现最终画出来的圆弧外边少了一部分，少的一部分是mBorderWidth/2,如图中的方形的框就是我们的View的大小，这个大小已经死定了的，我们要向让圆弧全部都显露出来就需要把里面画的区域缩小，这样才能全部显露出来。所以将RectF 缩小 mBorderWidth/2

2. 画出来的两边是条直线很不美观，需要将其改为圆形边框 pan

   ```java
    private void initPaints() {
           mOutPaint=new Paint();
           //设置抗锯齿
           mOutPaint.setAntiAlias(true);
           mOutPaint.setStrokeWidth(mBorderWidth);
           mOutPaint.setColor(mOutColor);
           //设置边缘
           mOutPaint.setStrokeCap(Paint.Cap.ROUND);
           //设置空心
           mOutPaint.setStyle(Paint.Style.STROKE);
       }
   ```

# 画内圆

内圆的画法跟外圆不一样了，内圆就需要动态的随着步数的变化而变化。定义总共的步数和现在的步数。然后算出比例，根据比例算出应该画的角度。

~~~java
    //总共的   当前的
    private int mStepMax=100;
    private int currentStep=50;
        //画内圆
        float sweepAngle=(float)currentStep/mStepMax;
        canvas.drawArc(rectF,135,sweepAngle*270,false,mInnerPaint);

~~~

# 画文字

~~~java
 //画文字
        String setpText=currentStep+"";
        Rect bound=new Rect();
        mTextPaint.getTextBounds(setpText,0,setpText.length(),bound);
        int dx=getWidth()/2-(bound.right-bound.left)/2;
        //基线 baseLine
        Paint.FontMetricsInt fontMetricsInt=mTextPaint.getFontMetricsInt();
        int dy=(fontMetricsInt.bottom-fontMetricsInt.top)/2-fontMetricsInt.bottom;
        int baseLine=getWidth()/2+dy;
        canvas.drawText(setpText,dx,baseLine,mTextPaint);
~~~

# 增加动画让其动起来

~~~java

    public void setMaxStep(int maxStep){
        this.mStepMax=maxStep;
    }


    public void setCurrentStep(int currentStep){
        this.currentStep=currentStep;
        //重新绘制
        invalidate();
    }
~~~

~~~java
public class MainActivity extends AppCompatActivity {

    ObjectAnimator objectAnimator;
    ValueAnimator valueAnimator;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        final QQStepView qqStepView=findViewById(R.id.stepView);
        qqStepView.setMaxStep(4000);
        valueAnimator=ObjectAnimator.ofInt(0,2000);
        valueAnimator.setInterpolator(new AccelerateInterpolator());
        valueAnimator.setDuration(2000);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                int value= (int) animation.getAnimatedValue();
                qqStepView.setCurrentStep(value);

            }
        });
        valueAnimator.start();

    }
}

~~~

