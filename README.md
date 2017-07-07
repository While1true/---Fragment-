# ---Fragment-
你所不知道的fragment的妙用，解放麻烦的OnActivityResult
### 你所不知道的Fragment的妙用
![pic](http://upload-images.jianshu.io/upload_images/6456519-1eb8c8d4884d9403.gif?imageMogr2/auto-orient/strip)





- 在软件开发中登陆功能是十分常见重要的，就以此为例说明fragment的一种用法，让开发变得更自如

#### 1.这个用法的原因和意义
1. 在未登录情况下，点击很多地方都可能要跳到登陆界面，登陆成功后，当前页面需要刷新
1.  我们的一般做法是StartActivityForResult，在登陆成功后，SetResultOK，finsh登陆页面。 在当前Activity或者fragment的OnActivityResult中根据RequestCode和ResultCode来判断登陆是否成功来刷新界面
1. 由于要登陆的地方很多，重复在不同的Activity或Fragment中重写OnactivityResult是个很头痛的事，甚至有时后我们在adapter或者dialog等中要获取这个登陆的回调是很麻烦的，数据要传来传去，用EVenbus也不能简化我们的操作
2. 有没有办法让我们，像设置OnClickListener这样直接获得登陆回调？
#### 2.先看看用法
在任何需要登陆的地方调用我们在Utils中写的一个静态方法
一次写好从此轻松调用

```
   ActivityUtils.startLogin(this, new ActivityUtils.ActivityResultListner() {
            @Override
            public void loginsuccess() {
                ((Button)v).setText("您已登陆");
            }

            @Override
            public void logincancel() {
                Toast.makeText(MainActivity.this,"您已取消登陆",0).show();
            }
        });
```
#### 3.实现的方式
- 利用fragment拥有和activity同步的生命周期
- frament中可以收到OnActivityResult的回调
1. 定义回调接口

```
//用abstract class我们在不需要cancel回掉时可以不重写
   public abstract static class ActivityResultListner {
        public void loginsuccess() {
        }

        public void logincancel() {
        }
    }
```

2.先定义一个fragment用于处理回调
```
 public static class MyFragment extends Fragment {

        public static final int LOGIN = 123;

        ActivityResultListner listener;

        public void setListener(ActivityResultListner listener) {
            this.listener = listener;
        }

        @Override
        public void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if (requestCode == LOGIN) {
                if (resultCode == RESULT_OK) {
                    if (listener != null)
                        listener.loginsuccess();
                } else {
                    if (listener != null)
                        listener.logincancel();
                }
            }
        }
        
```
3. 回调的实现
- 传进的Context 要是FragmentActivity的子类 
实际中
AppcomatActivity Fragment.getActivity()都是满足这个要求的
- 先把frament添加到activity
- 根据fragment的onActivityResult获取回调

```
   public static void startLogin(FragmentActivity context, ActivityResultListner listener) {
        //先看activity是否添加过该fragment， 添加根据Tag找出 ，没有就添加
        FragmentManager manager = context.getSupportFragmentManager();
        MyFragment myFragment = null;
        Fragment loginf = manager.findFragmentByTag(MyFragment.LOGIN + "");
        if (loginf == null) {
            myFragment = new MyFragment();
            manager.beginTransaction().add(myFragment, MyFragment.LOGIN + "").commit();
            //这句是让commit立即生效，不然运行会报错，fragment还没有被attach
            manager.executePendingTransactions();
        } else {
            myFragment = (MyFragment) loginf;
        }
        //设置监听
        myFragment.setListener(listener);
        Intent intent = new Intent(context, loginActivity.class);
        myFragment.startActivityForResult(intent, MyFragment.LOGIN);
    }
```

4.总结
-  利用了fragment和activity相同的生命周期，用同样的方法可以做很多事情，比如申请权限等
-  第一麻烦从此轻松调用，希望对各位有所帮助！



 
