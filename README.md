## Android MVVM ##

#### Android框架 ####

如今，Android框架日益发展，从MVC到MVP、MVVM

- MVC(Model-View-Controller)自不必说，大家应该都早已知道，在Android中，由于Activity层即承担View的职责，又有Controller职责，导致Activity异常臃肿，测试难，维护难。

- MVP(Model-View-Presenter)是从MVC演化而来，将View层与Model层解耦，Activity层作为View，只与Presenter层进行交互，Presenter通过接口对View进行操作，而Model层也只与Presenter交互，View层和Model层不进行直接交互，Presenter作为两者的桥梁。如此一来三者各司其职，降低耦合，又便于测试，便于维护。

- MVVM(Model-View-ViewModel)最早是由微软提出的，在Android中，MVVM和MVP比较相似，但它的核心在于`DataBinding`，View的变化可以自动的反映在ViewModel，ViewModel的数据变化也会自动反应到View上。这样我们就不用处理接收时间和View更新的工作，框架已经做好了。

MVC、MVP、MVVM优劣对比

- MVC：其实Android本身还是符合MVC架构的，但是Activity又是View又是Controller，导致Activity臃肿，高度耦合，动辄千行代码，维护测试困难。

- MVP：
	
	优势：MVP对于MVC做了优化，V层与P层通过接口交互，Activity只负责UI变化，业务逻辑放在P层之中，M层与V层不能直接访问，M与V层解耦。它的结构也更清晰了，每层都有它对应的职责，这样的结构便于维护与单元测试。
	
	劣势：以为V层与P层通过接口交互，定义接口就成为了一个问题，粒度太小，就会导致接口过多，对应Activity代码也会很多，粒度太大，解耦效果就差。且其本身虽然通过接口访问进行解耦，但是如果控件变更，对应的逻辑也要做相应的变更

- MVVM：万事万物皆有利弊
	
	优势：
	1. 低耦合，在MVVM中，数据是独立于UI的，数据和业务逻辑是在独立的ViewModel中，不涉及任何UI相关的事，不持有UI控件的引用，因此逻辑只需要关心数据即可。
	2. MVVM同样是便于维护就单元测试的，但是由于MVVM的ViewModel并不像MVP的Presenter一样是纯java代码，因此测试框架选择不同。
	
	劣势：
	1. 结构相对复杂
	2. 数据绑定是的bug很难被调试。当我们看到界面异常的时候，可能是View的代码有bug，也可能是Model的代码有问题。数据绑定让一个位置的bug快速传递到其他位置，要定义到原始出问题的地方就不那么容易了。
	3. 对于过大的项目，数据绑定要话费更大的内存。


#### Android DataBinding ####

DataBinding是google搞出来的数据绑定框架，是解决界面逻辑的一个黑科技。

DataBinding已经有很多大牛写过相关的文档，优美的文笔，严谨的逻辑，我是自愧不如，就不再多做更多的介绍，只在此推荐如下一篇文章。结合DataBinding官方文档，相信大家可以快速入门、精通。

DataBinding入门：[Android MVVM到底是啥？看完就明白了](http://mp.weixin.qq.com/s?__biz=MzA4MjU5NTY0NA==&mid=401410759&idx=1&sn=89f0e3ddf9f21f6a5d4de4388ef2c32f#rd)

DataBinding官方文档：[https://developer.android.com/topic/libraries/data-binding/index.html#build_environment](https://developer.android.com/topic/libraries/data-binding/index.html#build_environment)

#### MVVM实践 ####

Activity中绑定View与ViewModel

ViewModel中持有Model的引用，并且通过DataBinding绑定在View上，当data刷新时，view同步更新

PODEMO：登陆然后设置HOME界面List

包括两个页面，登陆页面和主页面

登陆页面分为LoginModel，LoginBean，LoginActivity，LoginViewModel

LoginModel中完成与服务器的交互，校验用户名密码的准确性

	/**
     * 模拟网络请求验证登录信息，实际应有回调
     * @param name 用户名
     * @param pwd 密码
     * @return 校验结果
     */
    public boolean checkLoginInfo(String name, String pwd) {
        return "vicky".equals(name) && "123".equals(pwd);
    }

LoginBean中包含用户名和密码

	private String loginName;
    private String loginPwd;

    public String getLoginName() {
        return loginName;
    }

    public void setLoginName(String loginName) {
        this.loginName = loginName;
    }

    public String getLoginPwd() {
        return loginPwd;
    }

    public void setLoginPwd(String loginPwd) {
        this.loginPwd = loginPwd;
    }

LoginActivity中绑定xml布局，并设置EditText的TextWatcher

	// DataBinding根据xml名称设置类，用以绑定xml
	private ActivityLoginBinding mBinding;
	// 持有ViewModel对象可以调用其中的方法
    private LoginViewModel mViewModel;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		// DataBinding绑定方式
        mBinding = DataBindingUtil.setContentView(this, R.layout.activity_login);
        mViewModel = new LoginViewModel(new LoginBean());
		// 绑定完成后要设置ViewModel
        mBinding.setLogin(mViewModel);
        attachListener();
    }

    private void attachListener() {
        mBinding.loginNameEdit.addTextChangedListener(mTextWatcher);
        mBinding.loginPwdEdit.addTextChangedListener(mTextWatcher);
    }

    private TextWatcher mTextWatcher = new TextWatcher() {
        @Override
        public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

        }

        @Override
        public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {

        }

        @Override
        public void afterTextChanged(Editable editable) {
            if (mBinding.loginNameEdit.getText().length() > 0
                    && mBinding.loginPwdEdit.getText().length() > 0) {
                mViewModel.setBtnVisible(true);
            } else {
                mViewModel.setBtnVisible(false);
            }
        }
    };

Login xml布局

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="login"
            type="com.daydayup.mvvm.viewmodel.LoginViewModel" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            android:orientation="horizontal">

            <TextView
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="10dp"
                android:layout_marginStart="10dp"
                android:layout_weight="1"
                android:gravity="end"
                android:paddingBottom="5dp"
                android:paddingTop="5dp"
                android:text="@string/login_name"
                android:textSize="15sp" />

            <EditText
                android:id="@+id/login_name_edit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="10dp"
                android:layout_marginStart="10dp"
                android:layout_weight="3"
                android:background="@null"
                android:hint="@string/input_login_name"
                android:inputType="text"
                android:paddingBottom="5dp"
                android:paddingTop="5dp"
                android:text="@={login.loginName}"
                android:textSize="15sp"/>

        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <TextView
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="10dp"
                android:layout_marginStart="10dp"
                android:layout_weight="1"
                android:gravity="end"
                android:paddingBottom="5dp"
                android:paddingTop="5dp"
                android:text="@string/login_pwd"
                android:textSize="15sp" />

            <EditText
                android:id="@+id/login_pwd_edit"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="10dp"
                android:layout_marginStart="10dp"
                android:layout_weight="3"
                android:background="@null"
                android:hint="@string/input_login_pwd"
                android:inputType="numberPassword"
                android:paddingBottom="5dp"
                android:paddingTop="5dp"
                android:text="@={login.loginPwd}"
                android:textSize="15sp" />

        </LinearLayout>

        <Button
            android:id="@+id/login_btn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginEnd="10dp"
            android:layout_marginStart="10dp"
            android:layout_marginTop="20dp"
            android:enabled="@{login.btnVisible}"
            android:onClick="@{login.handleLogin}"
            android:text="@string/login_btn_txt" />

    </LinearLayout>
	</layout>

LoginViewModel中设置获取用户名和密码，通过View的绑定显示这些信息在界面上，并制定登陆按钮是否可点击及点击时间

	// 持有Model对象，调用接口等
	private LoginModel mMode;
	// 显示并更新界面数据
    private LoginBean mBean;
	// 双向绑定，实时控制btn的Enable
    private boolean btnVisible;
	
	// 参数按照需要添加
    public LoginViewModel(LoginBean bean) {
        this.mBean = bean;
        mMode = new LoginModel();
    }

    public String getLoginName() {
        return mBean.getLoginName();
    }

    public void setLoginName(String loginName) {
        mBean.setLoginName(loginName);
    }

    public String getLoginPwd() {
        return mBean.getLoginPwd();
    }

    public void setLoginPwd(String loginPwd) {
        mBean.setLoginPwd(loginPwd);
    }

	// 双向绑定的写法 @Bindable及notifyPropertyChanged
    @Bindable
    public boolean isBtnVisible() {
        return btnVisible;
    }

    public void setBtnVisible(boolean btnVisible) {
        this.btnVisible = btnVisible;
        notifyPropertyChanged(BR.btnVisible);
    }

	// 处理btn的点击事件 
    public void handleLogin(View view) {
        Context context = view.getContext();
        if (mMode.checkLoginInfo(mBean.getLoginName(), mBean.getLoginPwd())) {
            context.startActivity(new Intent(context, HomeActivity.class));
        } else {
            Toast.makeText(context, "error name or pwd", Toast.LENGTH_SHORT).show();
        }
    }

主页面结构登录界面相同

HomeModel获取主页面列表

	// 模拟返回数据，此处应有回调
	public ArrayList<HomeBean> getList() {
        ArrayList<HomeBean> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            HomeBean homeBean = new HomeBean();
            homeBean.setDescription("Description:" + i);
            homeBean.setKeyWords("KeyWords:" + i);
            homeBean.setSummary("Summary:" + i);
            homeBean.setImg("http://img.bizhi.sogou.com/images/2012/03/14/140763.jpg");
            list.add(homeBean);
        }
        return list;
    }

HomeBean主页列表item(本demo只有列表，如有其它，可以定义第二个bean，如Login一样定义到ViewModel中)

	private String description;
    private String img;
    private String keyWords;
    private String summary;

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getImg() {
        return img;
    }

    public void setImg(String img) {
        this.img = img;
    }

    public String getKeyWords() {
        return keyWords;
    }

    public void setKeyWords(String keyWords) {
        this.keyWords = keyWords;
    }

    public String getSummary() {
        return summary;
    }

    public void setSummary(String summary) {
        this.summary = summary;
    }

	// ImageView绑定方法(PS：经测试"img"全工程共享)
    @BindingAdapter("img")
    public static void loadImg(ImageView imgView, String url) {
        Glide.with(imgView.getContext()).load(url).into(imgView);
    }

HomeActivity绑定xml，在onResume时刷新列表数据

	// 持有Model对象，调用接口等
	private HomeViewModel mViewModel;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityHomeBinding mBinding = DataBindingUtil.setContentView(this, R.layout.activity_home);
		mViewModel = new HomeViewModel();
        mBinding.setHome(mViewModel);
    }

    @Override
    protected void onResume() {
        super.onResume();
        mViewModel.refreshList();
    }

Home xml布局

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="home"
            type="com.daydayup.mvvm.viewmodel.HomeViewModel" />
    </data>

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <ListView
            android:id="@+id/home_list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:onItemClick="@{home.onItemClick}"
            app:itemView="@{home.itemView}"
            app:items="@{home.items}" />

    </LinearLayout>

	</layout>

Home item xml布局

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="homeItem"
            type="com.daydayup.mvvm.model.HomeBean" />
    </data>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="96dp">

        <ImageView
            android:id="@+id/iv"
            android:layout_width="96dp"
            android:layout_height="96dp"
            android:contentDescription="@null"
            android:padding="6dp"
            app:img="@{homeItem.img}" />

        <TextView
            android:id="@+id/list_view_description"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:layout_marginStart="8dp"
            android:layout_toEndOf="@id/iv"
            android:layout_toRightOf="@id/iv"
            android:ellipsize="end"
            android:text="@{homeItem.description}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentBottom="true"
            android:layout_marginBottom="2dp"
            android:layout_marginLeft="8dp"
            android:layout_marginStart="8dp"
            android:layout_toEndOf="@id/iv"
            android:layout_toRightOf="@id/iv"
            android:text="@{homeItem.keyWords}" />
    </RelativeLayout>

	</layout>

HomeViewModel显示在界面上的数据定义及item点击事件

	private HomeModel mModel;

	// list item(列表的bean对象如此加载)
    public final ObservableList<HomeBean> items = new ObservableArrayList<>();
	// item view
    public final ItemView itemView = ItemView.of(BR.homeItem, R.layout.item_list_view);

    public HomeViewModel() {
        mModel = new HomeModel();
    }

    public void refreshList() {
        items.clear();
        items.addAll(mModel.getList());
    }

	// item点击事件
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        Context context = view.getContext();
        Toast.makeText(context, "It's " + position + " item.", Toast.LENGTH_SHORT).show();
    }

简易DEMO，仅供参考！

Over.

参考：

1. [认清Android框架 MVC，MVP和MVVM](http://blog.csdn.net/jdsjlzx/article/details/51174396)
2. [Android MVVM到底是啥？看完就明白了](http://mp.weixin.qq.com/s?__biz=MzA4MjU5NTY0NA==&mid=401410759&idx=1&sn=89f0e3ddf9f21f6a5d4de4388ef2c32f#rd)
3. [Android数据绑定框架DataBinding](http://www.jianshu.com/p/2d3227d9707d)
4. [如何构建Android MVVM 应用框架](http://tech.meituan.com/android_mvvm.html?utm_source=tuicool&utm_medium=referral)
5. [HTML特殊转义字符对照表](http://tool.oschina.net/commons?type=2)
