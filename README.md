### Custom Notification Badge

For Creating badge:

**BadgeDrawable.java**

    `package com.example.priyankam.customnotificationbadge;

     import android.content.Context;
     import android.graphics.Canvas;
     import android.graphics.Color;
     import android.graphics.ColorFilter;
     import android.graphics.Paint;
     import android.graphics.Rect;
     import android.graphics.Typeface;
     import android.graphics.drawable.Drawable;

    public class BadgeDrawable extends Drawable {

    private float mTextSize;
    private Paint mBadgePaint;
    private Paint mTextPaint;
    private Rect mTxtRect = new Rect();

    private String mCount = "";
    private boolean mWillDraw = false;

    public BadgeDrawable(Context context) {
        // mTextSize = 10dp;
        mTextSize = context.getResources().getDimension(R.dimen.margin_ssmall);

        mBadgePaint = new Paint();
        mBadgePaint.setColor(Color.RED);
        mBadgePaint.setAntiAlias(true);
        mBadgePaint.setStyle(Paint.Style.FILL);

        mTextPaint = new Paint();
        mTextPaint.setColor(Color.WHITE);
        mTextPaint.setTypeface(Typeface.DEFAULT_BOLD);
        mTextPaint.setTextSize(mTextSize);
        mTextPaint.setAntiAlias(true);
        mTextPaint.setTextAlign(Paint.Align.CENTER);
    }

    @Override
    public void draw(Canvas canvas) {
        if (!mWillDraw) {
            return;
        }

        Rect bounds = getBounds();
        float width = bounds.right - bounds.left;
        float height = bounds.bottom - bounds.top;

        // Position the badge in the top-right quadrant of the icon.
        float radius = ((Math.min(width, height) / 2) - 1) / (float) 1.6;
        float centerX = width - radius - 1;
        float centerY = radius + 1;

        // Draw badge circle.
        canvas.drawCircle(centerX, centerY, radius, mBadgePaint);

        // Draw badge count text inside the circle.
        mTextPaint.getTextBounds(mCount, 0, mCount.length(), mTxtRect);
        float textHeight = mTxtRect.bottom - mTxtRect.top;
        float textY = centerY + (textHeight / 2f);
        canvas.drawText(mCount, centerX, textY, mTextPaint);
    }

    /*
    Sets the count (i.e notifications) to display.
     */
    public void setCount(int count) {
        mCount = Integer.toString(count);

        // Only draw a badge if there are notifications.
        mWillDraw = count > 0;
        invalidateSelf();
    }

    @Override
    public void setAlpha(int alpha) {
        // do nothing
    }

    @Override
    public int getOpacity() {
        // TODO Auto-generated method stub
        return 0;
    }


    @Override
    public void setColorFilter(ColorFilter cf) {
        // TODO Auto-generated method stub

    }
    }`

 **menu.xml**
    
    `<?xml version="1.0" encoding="utf-8"?>
     <menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
      <item
        android:id="@+id/action_cart"
        android:icon="@drawable/ic_menu_notifications"
        android:title="notification"
        app:showAsAction="always" />
    </menu>`

** In MainActivity.java added following code **

    `@Override
     public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        // Handle presses on the action bar items
        switch (id) {
            case R.id.action_cart:

                GlobalClass.setAddToCart(context, item,numMessages);
                invalidateOptionsMenu();

                //This is the proper solution for Open Some Activity  after clicking the action in notification
                // Open InfoActivity Class on "Info" Button Click
                Intent intentInfo = new Intent(this, InfoActivity.class);
                // Send data to InfoActivity Class
                intentInfo.putExtra("title", strTitle);
                intentInfo.putExtra("text", strText);
                startActivity(intentInfo);

                numMessages=0;//clear notification count
                invalidateOptionsMenu();

                return true;

            default:
                onBackPressed();
                return true;
        }

    }`


And also add the following code ,where you want to update the notification count:

    `bcustomnotify.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                GlobalClass.setNotifyCount(numMessages);
                // force the ActionBar to relayout its MenuItems.
                // onCreateOptionsMenu(Menu) will be called again.
                invalidateOptionsMenu();

            }
        });`

and also refresh the count onResume():

    `@Override
     protected void onResume() {
        super.onResume();
        // force the ActionBar to relayout its MenuItems.
        // onCreateOptionsMenu(Menu) will be called again.
        invalidateOptionsMenu();
    }`

**GlobalClass.java**
    
    `package com.example.priyankam.customnotificationbadge;

     import android.app.Activity;
     import android.content.Context;
     import android.graphics.drawable.LayerDrawable;
     import android.view.MenuItem;

     public class GlobalClass extends Activity {
      private static LayerDrawable icon;
      public GlobalClass() {
        //constructor
     }
     public static void setAddToCart(Context context, MenuItem item, int numMessages) {
        icon = (LayerDrawable) item.getIcon();
        SetNotificationCount.setBadgeCount(context, icon,GlobalClass.setNotifyCount(numMessages));
     }
     public static int setNotifyCount(int numMessages) {
        int count=numMessages;
        return count;
    }
    }`
**SetNotificationCount.java**

    `package com.example.priyankam.customnotificationbadge;

     import android.content.Context;
     import android.graphics.drawable.Drawable;
     import android.graphics.drawable.LayerDrawable;

     public class SetNotificationCount {
     public static void setBadgeCount(Context context, LayerDrawable icon, int count) {
        BadgeDrawable badge;
        // Reuse drawable if possible
        Drawable reuse = icon.findDrawableByLayerId(R.id.ic_badge);
        if (reuse != null && reuse instanceof BadgeDrawable) {
            badge = (BadgeDrawable) reuse;
        } else {
            badge = new BadgeDrawable(context);
        }
        badge.setCount(count);
        icon.mutate();
        icon.setDrawableByLayerId(R.id.ic_badge, badge);
    }
    }`

# Output:
![First Screen](https://raw.githubusercontent.com/Priyanka-Mohanty/CustomNotificationBadge/gh-pages/Images/Screenshot_20160803-151305.png)
![Second Screen](https://raw.githubusercontent.com/Priyanka-Mohanty/CustomNotificationBadge/gh-pages/Images/Screenshot_20160803-151309.png)
![Third Screen](https://raw.githubusercontent.com/Priyanka-Mohanty/CustomNotificationBadge/gh-pages/Images/Screenshot_20160803-151313.png)
![Fourth Screen](https://raw.githubusercontent.com/Priyanka-Mohanty/CustomNotificationBadge/gh-pages/Images/Screenshot_20160803-151335.png)
![Fifth Screen](https://raw.githubusercontent.com/Priyanka-Mohanty/CustomNotificationBadge/gh-pages/Images/Screenshot_20160803-151403.png)
