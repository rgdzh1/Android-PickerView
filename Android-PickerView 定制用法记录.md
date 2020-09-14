> [项目](https://github.com/Bigkoo/Android-PickerView)
#### 英文日期
`Android-PickerView`没有月份用英文的控件,在`Issues`中有人贴出了[方法](https://github.com/Bigkoo/Android-PickerView/issues/575).继承`WheelView`实现:
```java
public class EnglishMonthWheelView extends WheelView {
    public EnglishMonthWheelView(Context context) {
        super(context);
    }

    public EnglishMonthWheelView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }


    @Override
    protected String getContentText(Object item) {
        if (item == null || TextUtils.isEmpty(item.toString())) {
            return "";
        } else if (item instanceof IPickerViewData) {
            return ((IPickerViewData) item).getPickerViewText();
        } else if (item instanceof Integer) {
            //如果为整形则最少保留两位数.
            int value = (int) item;
            String month = null;
            switch (value) {
                case 1:
                    month = "Jan";
                    break;
                case 2:
                    month = "Feb";
                    break;
                case 3:
                    month = "Mar";
                    break;
                case 4:
                    month = "Apr";
                    break;
                case 5:
                    month = "May";
                    break;
                case 6:
                    month = "Jun";
                    break;
                case 7:
                    month = "Jul";
                    break;
                case 8:
                    month = "Aug";
                    break;
                case 9:
                    month = "Sep";
                    break;
                case 10:
                    month = "Oct";
                    break;
                case 11:
                    month = "Nov";
                    break;
                case 12:
                    month = "Dec";
                    break;
            }
            return month;
        }
        return item.toString();
    }
}
```
使用的如下:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="44dp"
        android:background="@color/white">

        <Button
            android:id="@+id/btn_cancel"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:layout_alignParentTop="true"
            android:background="@android:color/transparent"
            android:padding="8dp"
            android:text="@string/canceled"
            android:textAllCaps="false"
            android:textColor="@color/color_cancel"
            android:textSize="14sp" />

        <Button
            android:id="@+id/tv_title_desc"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:layout_centerInParent="true"
            android:background="@android:color/transparent"
            android:gravity="center"
            android:textAllCaps="false"
            android:textColor="@color/colorPrimary"
            android:textSize="14sp"
            tools:text="请选择" />

        <Button
            android:id="@+id/btn_confirm"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:layout_alignParentTop="true"
            android:layout_alignParentRight="true"
            android:background="@android:color/transparent"
            android:padding="8dp"
            android:text="@string/confirm"
            android:textAllCaps="false"
            android:textColor="@color/colorPrimary"
            android:textSize="14sp" />

    </RelativeLayout>

    <!--此部分需要完整复制过去，删减或者更改ID会导致初始化找不到内容而报空-->
    <LinearLayout
        android:id="@+id/timepicker"
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:background="@android:color/white"
        android:gravity="center"
        android:orientation="horizontal">
        <!--该控件就是上面处理过的,专门用来展示英文月-->
        <com.bigkoo.pickerview.view.EnglishMonthWheelView
            android:id="@+id/month"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />

        <com.contrarywind.view.WheelView
            android:id="@+id/day"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />


        <com.contrarywind.view.WheelView
            android:id="@+id/year"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />
        ...
    </LinearLayout>
</LinearLayout>
```
```kotlin
fun ContactFragment.selectBirthDatePV(mTargetView: YSelectTextView) {
    activity?.apply {
        KeyboardUtils.hideSoftInput(this) //影藏软键盘
    }
    var btnCancel: Button? = null
    var btnConfirm: Button? = null
    val selectedDate = Calendar.getInstance();
    val startDate = Calendar.getInstance();
    val endDate = Calendar.getInstance();
    startDate.set(1965, 0, 1);
    endDate.set(2004, 11, 31);
    selectedDate.set(1990, 0, 1);
    val mBirthDateTPV = TimePickerBuilder(activity, object : OnTimeSelectListener {
        override fun onTimeSelect(date: Date?, p1: View?) {
            date?.apply {
                val mDisplayData = this.dateToEnglishDateStr()// 将Date日期转为英文Date日期
                mTargetView.content = mDisplayData
            }
        }
    })
        .setLayoutRes(R.layout.layout_pickerview_item_timer) {
            btnCancel = it.findViewById<Button>(R.id.btn_cancel)
            btnConfirm = it.findViewById<Button>(R.id.btn_confirm)
            it.findViewById<TextView>(R.id.tv_title_desc).text =
                resources.getString(R.string.date_of_birth)
        }
        .setRangDate(startDate, endDate)
        .setDate(selectedDate)
        .setTextXOffset(0, 0, 0, 40, 0, -40)
        .setLabel("", "", "", "", "", "")//默认设置为年月日时分秒
        .isDialog(true)
        .setItemVisibleCount(17)
        .build()
    mBirthDateTPV.show()
    btnConfirm?.setOnClickListener {
        mBirthDateTPV.returnData()
        mBirthDateTPV.dismiss()
    }
    btnCancel?.setOnClickListener {
        mBirthDateTPV.dismiss()
    }
}
```
#### Dialog模式调整位置
Activity重写了返回键操作,在不同Fragment中会弹出PV,此时返回键被重写了,无法在按返回键的时候隐藏PV,这个时候用Dialog模式可以很好的处理这个问题,但是当前PV里使用Dialog布局就会居中,想要布局再最底部就要改源码,阅读源码发现BasePickerView.createDialog()中创建的Dialog,下面是修改后的createDialog()方法.
```java
public void createDialog() {
    if (dialogView != null) {
        mDialog = new Dialog(context, R.style.custom_dialog2);
        mDialog.setCancelable(mPickerOptions.cancelable);//不能点外面取消,也不能点back取消
        mDialog.setContentView(dialogView);
        Window dialogWindow = mDialog.getWindow();
        if (dialogWindow != null) {
            dialogWindow.setWindowAnimations(R.style.picker_view_slide_anim);
            dialogWindow.setGravity(Gravity.CENTER);//可以改成Bottom
        }
        // 手动设置Dialog样式的PickerView在屏幕底部,并且宽度为屏幕宽度,这样可以PickerView可以自动接管返回按键事件
        WindowManager.LayoutParams attributes = dialogWindow.getAttributes();
        attributes.width = WindowManager.LayoutParams.MATCH_PARENT;
        attributes.gravity = Gravity.BOTTOM;
        dialogWindow.setAttributes(attributes);
        mDialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
            @Override
            public void onDismiss(DialogInterface dialog) {
                if (onDismissListener != null) {
                    onDismissListener.onDismiss(BasePickerView.this);
                }
            }
        });
    }
}
```