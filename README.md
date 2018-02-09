# MacVolumeView
仿Mac调节声音view

## 效果图
![](https://github.com/kuangxiaoguo0123/MacVolumeView/blob/master/screenshots/volume.gif)
## 实现
下面我们讲一下具体的实现逻辑。

- 自定义styleable
```
<declare-styleable name="VolumeView">
	<attr name="ball_color" format="color" />
	<attr name="left_color" format="color" />
	<attr name="right_color" format="color" />
	<attr name="ball_radius" format="dimension" />
	<attr name="start_value" format="dimension" />
</declare-styleable>
```
 ball_color：拖动小球的颜色
left_color：左边线条的颜色
 right_color：右边线条的颜色
ball_radius：小球的半径
start_value：音量默认初始值

-  VolumeView
```
/**
 * Created by kuangxiaoguo on 2016/11/3.
 */

public class VolumeView extends View {

    private static final float DEFAULT_WIDTH = 220;
    private static final float DEFAULT_HEIGHT = 50;
    private static final int DEFAULT_BALL_RADIUS = 10;
    private static final int START_VALUE = 0;
    private Paint mBallPaint;
    private Paint mLeftPaint;
    private Paint mRightPaint;
    private int mBallColor;
    private float mBallRadius;
    private int mStartValue;
    private TouchEnum state;
    private float downX;
    private int mWidth;
    private int endValue;
    private RectF mRectF;
    private int mLeftColor;
    private int mRightColor;

    public VolumeView(Context context) {
        this(context, null);
    }

    public VolumeView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public VolumeView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initAttrs(context, attrs, defStyleAttr);
    }

    private void initAttrs(Context context, AttributeSet attrs, int defStyleAttr) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.VolumeView, defStyleAttr, 0);
        mBallColor = typedArray.getColor(R.styleable.VolumeView_ball_color, Color.GREEN);
        mLeftColor = typedArray.getColor(R.styleable.VolumeView_left_color, Color.BLUE);
        mRightColor = typedArray.getColor(R.styleable.VolumeView_right_color, Color.LTGRAY);
        mBallRadius = typedArray.getDimension(R.styleable.VolumeView_ball_radius, dp2px(DEFAULT_BALL_RADIUS));
        mStartValue = (int) typedArray.getDimension(R.styleable.VolumeView_start_value, dp2px(START_VALUE));
        typedArray.recycle();
        initPaint();
    }

    private void initPaint() {
        mBallPaint = new Paint();
        mLeftPaint = new Paint();
        mRightPaint = new Paint();
        initPaint(mBallPaint, mBallColor);
        initPaint(mLeftPaint, mLeftColor);
        initPaint(mRightPaint, mRightColor);
        mLeftPaint.setStrokeWidth(mBallRadius / 2);
        mRightPaint.setStrokeWidth(mBallRadius / 2);
    }

    private void initPaint(Paint paint, int color) {
        paint.setAntiAlias(true);
        paint.setColor(color);
        paint.setStyle(Paint.Style.FILL);
        paint.setStrokeCap(Paint.Cap.BUTT);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downX = event.getX();
                /**
                 * 点击小球
                 */
                if (downX > mStartValue && downX < mStartValue + 2 * mBallRadius) {
                    state = TouchEnum.TOUCH_BALL;
                } else {
                    /**
                     * 点击位置位于最左边小于球的直径处
                     */
                    if (downX < 2 * mBallRadius) {
                        downX = 0;
                    } else if (downX > endValue) {
                        /**
                         * 点击位置位于最右边
                         */
                        downX = endValue;
                    }
                    /**
                     * 使得mStartValue为我们按下的值刷新view
                     * 并把state的状态置为TouchEnum.TOUCH_BALL
                     */
                    mStartValue = (int) downX;
                    invalidate();
                    state = TouchEnum.TOUCH_BALL;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                if (state == TouchEnum.TOUCH_BALL) {
                    float moveX = event.getX();
                    float moveDistance = moveX - downX;
                    mStartValue += moveDistance;
                    if (mStartValue < 0) {
                        mStartValue = 0;
                    } else if (mStartValue > endValue) {
                        mStartValue = endValue;
                    }
                    invalidate();
                    /**
                     * 每刷新一次view之后把moveX的值赋给downX，这样在下次又重新获取moveX，两者相减就是我们滑动的距离。
                     */
                    downX = moveX;
                }
                break;
        }
        return true;
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = getWidth();
        endValue = (int) (mWidth - 2 * mBallRadius);
        mRectF = new RectF(0, 0, mWidth, h);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        mBallPaint.setColor(getResources().getColor(R.color.colorGray));
        canvas.drawRoundRect(mRectF, getHeight() / 5, getHeight() / 5, mBallPaint);
        mBallPaint.setColor(mBallColor);
        int verticalCenter = getHeight() / 2;
        canvas.drawLine(0, verticalCenter, mStartValue, verticalCenter, mLeftPaint);
        canvas.drawCircle(mStartValue + mBallRadius, verticalCenter, mBallRadius, mBallPaint);
        canvas.drawLine(mStartValue + 2 * mBallRadius, verticalCenter, mWidth, verticalCenter, mRightPaint);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        if (widthMode == MeasureSpec.AT_MOST) {
            widthSize = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, DEFAULT_WIDTH, getResources().getDisplayMetrics());
        }
        if (heightMode == MeasureSpec.AT_MOST) {
            heightSize = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, DEFAULT_HEIGHT, getResources().getDisplayMetrics());
        }
        setMeasuredDimension(widthSize, heightSize);
    }

    public enum TouchEnum {
        TOUCH_BALL, TOUCH_LEFT, TOUCH_RIGHT
    }

    private int dp2px(int dp) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dp, getResources().getDisplayMetrics());
    }
}

```
-  xml使用VolumeView
```
 <com.asiatravel.macvolumeview.wediget.VolumeView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:left_color="#4A9AF9"
        app:right_color="#BBB9BB"
        app:ball_color="#fff"
        app:start_value="20dp" />
```
## Sample source code
[https://github.com/kuangxiaoguo0123/MacVolumeView](https://github.com/kuangxiaoguo0123/MacVolumeView)

## More information
[http://blog.csdn.net/kuangxiaoguo0123/article/details/53022639](http://blog.csdn.net/kuangxiaoguo0123/article/details/53022639)

