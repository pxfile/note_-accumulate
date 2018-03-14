MVC，MVP，MVVM
===


Android App的设计架构:MVC,MVP,MVVM与架构经验谈 和MVC框架模式 样，Model模型处 数据代码 变在Android的App开发中，很多 经常会头疼于App的架

构如何设计:

我的App需要应 这些设计架构吗?

MVC,MVP等架构讲的是 么?区别是 么? 本 就来带你分析 下这 个架构的特性，优缺点，以及App架构设计中应该注意的问题。

1.架构设计的 的

通过设计使程序模块化，做到模块内部的 聚合和模块之间的低耦合。这样做的好处是使得程序在开发的过
程中，开发 员只需要专注于 点，提 程序开发的效率，并且 容 进 后续的测试以及定位问题。但设
计 能违背 的，对于 同 级的 程，具体架构的实现 式必然是 同的，切忌犯为 设计 设计，为 
架构 架构的 病。 

举个简单的 :

个Android App如果只有3个Java 件，那只需要做点模块和层次的划分就可以，引 框架或者架构反 提 作 ，降低 产 ; 但如果当前开发的App最终代码 在10W 以上，本地需要进 复杂 操作，同时也需要考虑到与其余的Android开发者以及后台开发 员之间的同步配合，那就需要在架构 上进 些思考!

2.MVC设计架构

MVC简介

MVC全名是Model View Controller，如图，是模型(model)-视图(view)-控制 (controller)的缩写， 种软 件设计典范， 种业务逻辑、数据、界 显示分离的 法组织代码，在改进和个性化定制界 及 户交互 的同时， 需要重新编写业务逻辑。

其中M层处 数据，业务逻辑等;V层处 界 的显示结果;C层起到桥 的作 ，来控制V层和M层通信以 此来达到分离视图显示和业务逻辑层。

Android中的MVC Android中界 部分也采 当前 较流 的MVC框架，在Android中:

视图层(View) 般采 XML 件进 界 的描述，这些XML可以 解为AndroidApp的View。使 的时 候可以 常 的引 。同时 于后期界 的修改。逻辑中与界 对应的id 变化则代码 修改， 增强 代码的可维护性。

控制层(Controller) Android的控制层的重任通常落在 众多的Activity的肩上。这 话也就暗含 要在 Activity中写代码，要通过Activity交割Model业务逻辑层处 ，这样做的另外 个原因是Android中的 Actiivity的响应时间是5s，如果耗时的操作放在这 ，程序就很容 被回收掉。

模型层(Model) 我们针对业务模型，建 的数据结构和相关的类，就可以 解为AndroidApp的Model， Model是与View 关， 与业务相关的(感谢@Xander的讲解)。对数据库的操作、对 络等的操作都 应该在Model 处 ，当然对业务计算等操作也是必须放在的该层的。就是应 程序中 进制的数 据。

MVC代码实 我们来看看MVC在Android开发中是怎么应 的吧! 先上界 图

Controller控制 &View public class MainActivity extends ActionBarActivity implements OnWeatherListener, View.OnClickListener {

 private WeatherModel weatherModel;
 private EditText cityNOInput;
 private TextView city;
 ... 

 @Override
 protected void onCreate(Bundle savedInstanceState) { 

 super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main);
     weatherModel = new WeatherModelImpl(); 

|  |
| 

 initView();
} 

//初始化View
private void initView() {

 cityNOInput = findView(R.id.et_city_no);
    city = findView(R.id.tv_city);
    ...
    findView(R.id.btn_go).setOnClickListener(this); 

}

//显示结果 public void displayResult(Weather weather) {

 WeatherInfo weatherInfo = weather.getWeatherinfo();
    city.setText(weatherInfo.getCity());
    ... 

}

@Override
public void onClick(View v) { 

 switch (v.getId()) {
        case R.id.btn_go: 

;

} }

weatherModel.getWeather(cityNOInput.getText().toString().trim(), this)
break; 

 @Override
 public void onSuccess(Weather weather) { 

 displayResult(weather);
 } 

 @Override
 public void onError() { 

Toast.makeText(this, 获取天 信息失败, Toast.LENGTH_SHORT).show(); }

 private T findView(int id) {
     return (T) findViewById(id); 

}

} 从上 代码可以看到，Activity持有 WeatherModel模型的对象，当 户有点击Button交互的时候，Activity 作为Controller控制层读取View视图层EditTextView的数据，然后向Model模型发起数据请求，也就是调 WeatherModel对象的 法 getWeather() 法。当Model模型处 数据结束后，通过接 OnWeatherListener通知View视图层数据处 完毕，View视图层该 新界 UI 。然后View视图层调 displayResult() 法 新UI。 此，整个MVC框架流程就在Activity中体现出来 。

 |
|  |

|  |
| 

Model模型 来看看WeatherModelImpl代码实现 public interface WeatherModel { void getWeather(String cityNumber, OnWeatherListener listener); } ................

public class WeatherModelImpl implements WeatherModel { /这部分代码范 有问题， 络访问 应该在 Model中，应该把 络访问换成从数据库读取/ @Override public void getWeather(String cityNumber, final OnWeatherListener listener) {

/*数据层操作*/

 VolleyRequest.newInstance().newGsonRequest(http://www.weather.com.cn/data/sk/
 + cityNumber + .html, 

}

Weather.class, new Response.Listener<weather>() {
    @Override 

 public void onResponse(Weather weather) {
        if (weather != null) { 

 listener.onSuccess(weather);
        } else { 

 listener.onError();
        } 

 }
}, new Response.ErrorListener() { 

 @Override
    public void onErrorResponse(VolleyError error) { 

 listener.onError();
    } 

});

} 以上代码看出，这 设计 个WeatherModel模型接 ，然后实现 接 WeatherModelImpl类。controller 控制 activity调 WeatherModelImpl类中的 法发起 络请求，然后通过实现OnWeatherListener接 来获 得 络请求的结果通知View视图层 新UI 。 此，Activity就将View视图显示和Model模型数据处 隔离开 。activity担当contronller完成 model和view之间的协调作 。

于这 为 么 直接设计成类 的 个getWeather() 法直接请求 络数据?你考虑下这种情况:现 在代码中的 络请求是使 Volley框架来实现的，如果哪天 板 要你使 Afinal框架实现 络请求，你怎么 解决问题?难道是修改 getWeather() 法的实现? no no no，这样修改 仅破坏 以前的代码， 且还 于维护， 考虑到以后代码的扩展和维护性，我们选择设计接 的 式来解决着 个问题，我们实现另外 个WeatherModelWithAfinalImpl类，继承 WeatherModel，重写 的 法，这样 仅保 以前的 WeatherModelImpl类请求 络 式，还增加 WeatherModelWithAfinalImpl类的请求 式。Activity调 代码 需要任何修改。

3.MVP设计架构

 |
|  |

|  |
| 

在App开发过程中，经常出现的问题就是某 部分的代码 过 ，虽然做 模块划分和接 隔离，但也很难 完全避免。从实践中看到，这 多的出现在UI部分，也就是Activity 。想象 下， 个2000+ 以上基本 带注释的Activity，我的第 反应就是想吐。Activity内容过多的原因其实很好解释，因为Activity本身需要担负 与 户之间的操作交互，界 的展示， 是单纯的Controller或View。 且现在 部分的Activity还对整个App 起到类似iOS中的【ViewController】的作 ，这 带 的逻辑代码，造成Activity的臃肿。为 解决这 个问题，让我们引 MVP框架。

MVC的缺点

在Android开发中，Activity并不是一个标准的MVC模式中的Controller，它的 要职责是加载应 的布局和初始化 户 界 ，并接受并处 来 户的操作请求，进 作出响应。随着界 及其逻辑的复杂度 断提升， Activity类的职责 断增加，以致变得庞 臃肿。

么是MVP? MVP从 早的MVC框架演变过来，与MVC有 定的相似性:Controller/Presenter负责逻辑的处 ，Model提

供数据，View负责显示。

MVP框架由3部分组成:View负责显示，Presenter负责逻辑处 ，Model提供数据。在MVP模式 通常包含 3个要素(加上View interface是4个):

View:负责绘制UI元素、与 户进 交互(在Android中体现为Activity) Model:负责存储、检索、操纵数据(有时也实现 个Model interface 来降低耦合) Presenter:作为View与Model交互的中间纽带，处 与 户交互的负责逻辑。

*View interface:需要View实现的接 ，View通过View interface与Presenter进 交互，降低耦合， 进

 |
|  |

|  |
| 

单元测试

Tips:*View interface的必要性

回想 下你在开发Android应 时是如何对代码逻辑进 单元测试的?是否每次都要将应 部署到Android模 拟 或真机上，然后通过模拟 户操作进 测试?然 由于Android平台的特性，每次部署都耗费 的 时间，这直接导致开发效率的降低。 在MVP模式中，处 复杂逻辑的Presenter是通过interface与 View(Activity)进 交互的，这说明我们可以通过 定义类实现这个interface来模拟Activity的 为对Presenter 进 单元测试，省去 的部署及测试的时间。

MVC → MVP

当我们将Activity复杂的逻辑处 移 另外的 个类(Presenter)中时，Activity其实就是MVP模式中的 View，它负责UI元素的初始化，建 UI元素与Presenter的关联(Listener之类)，同时 也会处 些简 单的逻辑(复杂的逻辑交由 Presenter处 )。

MVP的Presenter是框架的控制者，承担 的逻辑操作， MVC的Controller 多时候承担 种转发的作 。因此在App中引 MVP的原因，是为 将此前在Activty中包含的 逻辑操作放到控制层中，避免 Activity的臃肿。

两种模式的主要区别:

(最主要区别)View与Model并 直接交互， 是通过与Presenter交互来与Model间接交互。 在MVC 中View可以与Model直接交互 通常View与Presenter是 对 的，但复杂的View可能绑定多个Presenter来处 逻辑。 Controller是基 于 为的，并且可以被多个View共享，Controller可以负责决定显示哪个View Presenter与View的交互是通过接 来进 的， 有 于添加单元测试。 MVC与MVP区别

因此我们可以发

现MVP的优点如下:

 |
|  |

1、模型与视图完全分离，我们可以修改视图 影响模型; 2、可以 效地使 模型，因为所有的交互都发 在 个地 ——Presenter内部;

3、我们可以将 个Presenter 于多个视图， 需要改变Presenter的逻辑。这个特性 常的有 ，因为视 图的变化总是 模型的变化频繁;

4、如果我们把逻辑放在Presenter中，那么我们就可以脱离 户接 来测试这些逻辑(单元测试)。 具体到Android App中， 般可以将App根据程序的结构进 纵向划分，根据MVP可以将App分别为模型层

(M)，UI层(V)和逻辑层(P)。

UI层 般包括Activity，Fragment，Adapter等直接和UI相关的类，UI层的Activity在启动之后实 化相应的 Presenter，App的控制权后移，由UI转移到Presenter，两者之间的通信通过BroadCast、Handler或者接 完成，只传递事件和结果。

举个简单的 ，UI层通知逻辑层(Presenter) 户点击 个Button，逻辑层(Presenter) 决定应该 么 为进 响应，该找哪个模型(Model)去做这件事，最后逻辑层(Presenter)将完成的结果 新到 UI层。

MVP的变种:Passive View

MVP的变种有很多，其中使 最 泛的是Passive View模式，即被动视图。在这种模式下，View和Model之 间 能直接交互，View通过Presenter与Model打交道。Presenter接受View的UI请求，完成简单的UI处 逻 辑，并调 Model进 业务处 ，并调 View将相应的结果反映出来。View直接依赖Presenter，但是 Presenter间接依赖View，它直接依赖的是View实现的接 。

相对于View的被动，那Presenter就是主动的 。对于Presenter的主动，有如下的 解:

Presenter是整个MVP体系的控制中 ， 是单纯的处 View请求的 ; View仅仅是 户交互请求的汇报者，对于响应 户交互相关的逻辑和流程，View 参与决策，真正的决 策者是Presenter; View向Presenter发送 户交互请求应该采 这样的 吻:“我现在将 户交互请求发送给你，你看着 办，需要我的时候我会协助你”， 应该是这样:“我现在处 户交互请求 ，我知道该怎么办，但是 我需要你的 持，因为实现业务逻辑的Model只信任你”; 对于绑定到View上的数据， 应该是View从Presenter上“拉”回来的，应该是Presenter主动“推”给View

的;

View尽可能 维护数据状态，因为其本身仅仅实现单纯的、独 的UI操作;Presenter才是整个体系的协 调者，它根据处 于交互的逻辑给View和Model安排 作。

MVP架构存在的问题与解决办法 加 模板 法(Template Method) 转移逻辑操作之后可能部分较为复杂的Activity内代码 还是 少，

于是需要在分层的基础上再加 模板 法(Template Method)。

具体做法是在Activity内部分层。其中最顶层为BaseActivity， 做具体显示， 是提供 些基础样式， Dialog，ActionBar在内的内容，展现给 户的Activity继承BaseActivity，重写BaseActivity预 的 法。如有 必要再进 次继承，App中Activity之间的继承次数最多 超过3次。

Model内部分层 模型层(Model)中的整体代码 是最 的， 般由 的Package组成，针对这部分 需要做的就是在程序设计的过程中，做好模块的划分，进 接 隔离，在内部进 分层。

强化Presenter 强化Presenter的作 ，将所有逻辑操作都放在Presenter内也容 造成Presenter内的代 码 过 ，对于这点，有 个 法是在UI层和Presenter之间设置中介者Mediator，将 如数据校验、组 装在内的轻 级逻辑操作放在Mediator中;在Presenter和Model之间使 代 Proxy;通过上述两者分 担 部分Presenter的逻辑操作，但整体框架的控制权还是在Presenter 中。Mediator和Proxy 是必须 的，只在Presenter负担过 时才建议使 。

最终的架构如下图所示:

MVP代码实 我们来看看MVP在Android开发中是怎么应 的吧!! 我们 另 个 来解释。
先来看包结构图

建 Bean

public class UserBean { private String mFirstName; private String mLastName; public UserBean(String firstName, String lastName) { this. mFirstName = firstName; this. mLastName = lastName; } public String getFirstName() { return mFirstName; } public String getLastName() { return mLastName; } }

建 Model (处 业务逻辑，这 指数据读写)，先写接 ，后写实现 public interface IUserModel { void setID(int id);

void setFirstName(String firstName);
void setLastName(String lastName);
int getID();
UserBean load(int id);// 通过id读取user信息,返回 个UserBean

} 实现 在这 写 Presenter控制

建 presenter(主导 ，通过iView和iModel接 操作model和view)，activity可以把所有逻辑给presenter处 ，这样java逻辑就从 机的activity中分离出来。 public class UserPresenter { private IUserView mUserView; private IUserModel mUserModel;

 public UserPresenter(IUserView view) {
        mUserView = view; 

 mUserModel = new UserModel(); 

}

 public void saveUser( int id, String firstName, String lastName) {
        mUserModel.setID(id); 

 mUserModel.setFirstName(firstName);
        mUserModel.setLastName(lastName); 

}

 public void loadUser( int id) {
       UserBean user = mUserModel.load(id); 

mUserView.setFirstName(user.getFirstName()); // 通过调 IUserView的 法来 新 显示

}

}

mUserView.setLastName(user.getLastName()); 

View视图 建 view( 新ui中的view状态)，这 出需要操作当前view的 法，也是接 public interface IUserView { int getID();

 String getFristName();
  String getLastName();
  void setFirstName(String firstName);
  void setLastName(String lastName); 

}
activity中实现iview接 ，在其中操作view，实 化 个presenter变 。 public class MainActivity extends Activity implements OnClickListener,IUserView {

 UserPresenter presenter;
  EditText id,first,last;
  @Override
  protected void onCreate(Bundle savedInstanceState) { 

 super.onCreate(savedInstanceState);
        setContentView(R.layout. activity_main); 

|  |
| 

}

}

findViewById(R.id. save).setOnClickListener( this);
findViewById(R.id. load).setOnClickListener( this); 

 id = (EditText) findViewById(R.id. id);
 first = (EditText) findViewById(R.id. first);
 last = (EditText) findViewById(R.id. last); 

 presenter = new UserPresenter( this); 

@Override
public void onClick(View v) { 

 switch (v.getId()) {
 case R.id. save: 

 presenter.saveUser(getID(), getFristName(), getLastName()); 

 break;
 case R.id. load: 

 presenter.loadUser(getID()); 

 break;
 default: 

break; }

@Override
public int getID() { 

 return new Integer( id.getText().toString()); 

}

@Override
public String getFristName() { 

 return first.getText().toString(); 

}

@Override
public String getLastName() { 

 return last.getText().toString(); 

}

@Override
public void setFirstName(String firstName) { 

 first.setText(firstName); 

}

@Override
public void setLastName(String lastName) { 

}

last.setText(lastName); 

 |
|  |

} 因此，Activity及从MVC中的Controller中解放出来 ，这会Activity主要做显示View的作 和 户交互。每 个Activity可以根据 显示View的 同实现View视图接 IUserView。

通过对 同 实 的MVC与MVP的代码，可以证实MVP模式的 些优点:

在MVP中，Activity的代码 臃肿; 在MVP中，Model(IUserModel的实现类)的改动 会影响Activity(View)，两者也互 涉， 在MVC中 会;
在MVP中，IUserView这个接 可以实现 地对Presenter的测试; 在MVP中，UserPresenter可以 于多个视图，但是在MVC中的Activity就 。

4.MVC、MVP与MVVM的关系 先介绍下MVVM。

MVVM

MVVM可以算是MVP的升级版，其中的VM是ViewModel的缩写，ViewModel可以 解成是View的数据模型和 Presenter的合体，ViewModel和View之间的交互通过Data Binding完成， Data Binding可以实现双向的交 互，这就使得视图和控制层之间的耦合程度进 步降低，关注点分离 为彻底，同时减轻 Activity的压 。

在 较之前，先从图上看看三者的异同。 

刚开始 解这些概念的时候认为这 种模式虽然都是要将view和model解耦，但是 此即彼，没有关系， 个应 只会 种模式。后来慢慢发现世界绝对 是只有 两 ，中间最 的 块其实是灰 地带，同 样，这 种模式的边界并 那么明显，可能你在 的应 中都会 到。实际上也根本没必要去纠结 到 底 的是MVC、MVP还是MVVP， 管 猫 猫，捉住 就是好猫。

MVC->MVP->MVVM演进过程

MVC -> MVP -> MVVM 这 个软件设计模式是 步步演化发展的，MVVM 是从 MVP 的进 步发展与规范， MVP 隔离 MVC中的 M 与 V 的直接联系后，靠 Presenter 来中转，所以使 MVP 时 P 是直接调 View 的接 来实现对视图的操作的，这个 View 接 的东 般来说是 showData、showLoading等等。M 与 V已

经隔离 ， 测试 ，但代码还 够优雅简洁，所以 MVVM 就弥补 这些缺陷。在 MVVM 中就出现的 Data Binding 这个概念，意思就是 View 接 的 showData 这些实现 法可以 写 ，通过 Binding 来实现。

同

如果把这三者放在 起 较，先说 下三者的共同点，也就是Model和View: Model:数据对象，同时，提供本应 外部对应 程序数据的操作的接 ，也可能在数据变化时发出变 通

知。Model 依赖于View的实现，只要外部程序调 Model的接 就能够实现对数据的增删改查。 View:UI层，提供对最终 户的交互操作功能，包括UI展现代码及 些相关的界 逻辑代码。 异 三者的差异在于如何粘合View和Model，实现 户的交互操作以及变 通知

Controller Controller接收View的操作事件，根据事件 同，或者调 Model的接 进 数据操作，或者 进 View的跳转，从 也意味着 个Controller可以对应多个View。Controller对View的实现 太关 ， 只会被动地接收，Model的数据变 通过Controller直接通知View，通常View采 观察者模式监听 Model的变化。

Presenter Presenter与Controller 样，接收View的命令，对Model进 操作;与Controller 同的是 Presenter会反作 于View，Model的变 通知 先被Presenter获得，然后Presenter再去 新View。 个Presenter只对应于 个View。根据Presenter和View对逻辑代码分担的程度 同，这种模式 有两种 情况:Passive View和Supervisor Controller。

ViewModel 注意这 的“Model”指的是View的Model，跟MVVM中的 个Model 是 回事。所谓View的 Model就是包含View的 些数据属性和操作的这么 个东东，这种模式的关键技术就是数据绑定(data binding)，View的变化会直接影响ViewModel，ViewModel的变化或者内容也会直接体现在View上。这 种模式实际上是框架替应 开发者做 些 作，开发者只需要较少的代码就能实现 较复杂的交互。

点 得

MVP和MVVM完全隔离 Model和View，但是在有些情况下，数据从Model到ViewModel或者Presenter的拷 开销很 ，可能也会结合MVC的 式，Model直接通知View进 变 。在实际的应 中很有可能你已经在 知 觉中将 种模式融合在 起，但是为 代码的可扩展、可测试性，必须做到模块的解耦， 相关的代 码 要放在 起。 上有 个故事讲， 个 在 家公司做 个新产品时， 名外包公司的新员 直接在 View中做 数据库持久化操作， 且 个hibernate代码展开后发现竟然有 百 的SQL语 ，搞得他们惊讶 已， 时成为笑谈。

个 解，在 义地谈论MVC架构时，并 指本 中严格定义的MVC， 是指的MV*，也就是视图和模型的 分离，只要 个框架提供 视图和模型分离的功能，我们就可以认为它是 个MVC框架。在开发深 之后， 可以再体会 到的框架到底是MVC、MVP还是MVVM。

5\. 基于AOP的框架设计

AOP(Aspect-Oriented Programming, 向切 编程)，诞 于上个世纪90 代，是对OOP(Object-Oriented Programming, 向对象编程)的补充和完善。OOP引 封装、继承和多态性等概念来建 种对象层次结 构， 以模拟公共 为的 个集合。当我们需要为分散的对象引 公共 为的时候，OOP则显得 能为 。 也就是说，OOP允许你定义从上到下的关系，但并 适合定义从左到右的关系。 如 志功能。 志代码往 往 平地散布在所有对象层次中， 与它所散布到的对象的核 功能毫 关系。对于其他类型的代码，如安 全性、异常处 和透明的持续性也是如此。这种散布在各处的 关的代码被称为横切(Cross-Cutting)代 码，在OOP设计中，它导致 代码的重复， 于各个模块的重 。 AOP技术则恰恰相反，它 种称为“横切”的技术，剖解开封装的对象内部，并将那些影响 多个类的公共 为封装到 个可重 模 块，并将其名为“Aspect”，即 。所谓“  ”，简单地说，就是将那些与业务 关，却为业务模块所共同调 的逻辑或责任封装起来， 于减少系统的重复代码，降低模块间的耦合度，并有 于未来的可操作性和可 维护性。

5.1 AOP在Android中的使

AOP把软件系统分为两个部分:核 关注点和横切关注点。业务处 的主要流程是核 关注点，与之关系 的部分是横切关注点。横切关注点的 个特点是，他们经常发 在核 关注点的多处， 各处都基本相 似。AOP的作 在于分离系统中的各种关注点，将核 关注点和横切关注点分离开来。在Android App中，哪 些是我们需要的横切关注点?个 认为主要包括以下 个 :Http, SharedPreferences, Json, Xml, File, Device, System, Log, 格式转换等。Android App的需求差别很 ， 同的需求横切关注点必然是 样的。 般的App 程中应该有 个Util Package来存放相关的切 操作，在项 多 之后可以将其中使 较多的 Util封装为 个Jar包供 程调 。

在使 MVP和AOP对App进 纵向和横向的切割之后，能够使得App整体的结构 清晰合 ，避免局部的代 码臃肿， 开发、测试以及后续的维护。

6\. 货:AndroidApp架构的设计经验 先是作者最最喜欢的 话，也是对创业公司特别适 的 话，也是对 要过度设计的 种诠释: 先实现，再重构吧。直接考虑代码 臃肿得话， 知道 么时候才能写好 joy 先实现，再重构吧。直接考虑代码 臃肿得话， 知道 么时候才能写好 joy 先实现，再重构吧。直接考虑代码 臃肿得话， 知道 么时候才能写好 joy (重要的事情说三遍sunglasses)

6.1 整体架构

代码和 档规范，根据需求进 模块划分，确定交互 式，形成接 档，这些较为通 的内容 再细说。 做Android App时， 般将App进 纵向和横向的划分。纵向的App由UI层，逻辑层和模型层构成，整体结构 基于MVP思想(图 来 络)。

UI层内部多 模板 法，以Activity为 般有BaseActivity，提供包括 些基础样式，Dialog，ActionBar在 内的内容，展现的Activity都会继承BaseActivity并实现预 的接 ，Activity之间的继承 超过3次;为避免 Activity内代码过多，将App的整体控制权后移，也借鉴 IOC做法， 的逻辑操作放在逻辑层中，逻辑层 和UI层通过接 或者Broadcast等实现通信，只传递结果。 般Activity 的代码 都很 ，通过这两种 式 般我写的单个Activity内代码 超过400 。

逻辑层实现的是绝 部分的逻辑操作，由UI层启动，在内部通过接 调 模型层的 法，在逻辑层内 使

|  |
| 

代 。打个 ，UI层告诉逻辑层我需要做的事，逻辑层去找相应的 (模型层)去做，最后只告诉UI这 件事做的结果。

模型层没 么好说的，这部分 般由 的Package组成，代码 是三层中最 的，需要在内部进 分层。

横向的分割依据AOP 向切 的思想，主要是提取出共 法作为 个单独的Util，这些Util会在App整体中 穿插使 。很多 的App都会引 封装的Jar包，封装 包括 件、JSON、SharedPreference等在内的 常 操作， 写的 起来顺 ，也 幅度降低 重复作业。

这样纵，横两次对于App代码的分割已经能使得程序 会过多堆积在 个Java 件 ，但靠 次开发过程就 写出 质 的代码是很困难的，趁着项 的间歇期，对代码进 重构很有必要。

6.2 类库的使 现在有很多帮助快速开发的类库，活 这些类库也是避免代码臃肿和混乱的好 法，下 给题主推荐 个常

类库。

减少Activity代码 的依赖注 框架ButterKnife:

https://github.com/JakeWharton/butterknife

简化对于SQlite操作的对象关系映射框架OrmLite:

https://github.com/j256/ormlite-android

图 缓存类库Fresco(by facebook):

https://github.com/facebook/fresco

能 第三 库就 第三 库。别管是否稳定，是否被持续维护，因为，任何第三 库的作者，都能碾压刚 
 的菜 ，你绝对写 出 别  好的代码 。 

最后附上知乎上 点赞次数很 的 段话: 如果“从零开始”， 么设计架构的问题属于想得太多做得太少的问题。

从零开始意味着 个项 的主要技术难点是基本功能实现。当每 个功能都需要考虑如何做到的时候，我觉
得 般 都没办法考虑如何做好。 

因为，所有的优化都是站在最上层进 统筹规划。在这之前，你必须对下层的每 个模块都 常熟悉，进 
提炼可复 的代码、规划逻辑流程。 

所以，如果真的是从零开始，别想太多 

 |
|  |