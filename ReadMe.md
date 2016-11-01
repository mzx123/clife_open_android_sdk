#Android接入指南

注：本文为C-Life Android终端开发工具的新手使用教程，只涉及教授SDK的使用方法，默认读者已经熟悉IDE的基本使用方法（本文以Android Studio为例），以及具有一定的编程知识基础等。

##1. 向C-Life注册你的应用程序
请到 应用中心 页面创建移动应用，填写资料后，将获得AppID和AppSecret，可立即用于开发。但应用登记完成后还需要提交审核，只有审核通过的应用才能正式发布使用。

##2. 下载C-Life终端SDK文件
请前往[下载中心](http://open.clife.net/#/documentation/downloadCenter)下载最新SDK包。

##3. 搭建开发环境
###3.1在Android Studio中建立你的工程。
###3.2在工程中新建一个libs目录，将开发工具包中的:
   * HetOpenLib-release.aar
   * bindlogicllib-release.aar
   * UdpCore_v1.2-release.aar
   * BlueToothSupport-release.aar    
       
   复制到该目录中（如下图所示，把aar包复制到libs目录下）。
    ![](http://i.imgur.com/KfIJUyt.png)
    
    
   在gradle中加入以下内容:
   
	```
		repositories {
		 flatDir {
		   dirs 'libs' //this way we can find the .aar file in libs folder
		}
		compile(name:'HetOpenLib-release', ext:'aar')
		compile(name:'bindlogicllib-release', ext:'aar')
		compile(name:'UdpCore_v1.2-release', ext:'aar')
		compile(name:'BlueToothSupport-debug', ext:'aar')
	```
   
  

###3.3添加权限
   需在AndroidManifest.xml中声明以下权限
   
	```
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.READ_PHONE_STATE" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
	<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
	<uses-permission android:name="android.permission.BLUETOOTH" />
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
	```
	
###3.4添加授权登录页面的文件声明
   
	```
	<activity android:name="com.het.open.lib.ui.HetWebViewActivity" android:exported="true">
	</activity>
	```
	
##4. 功能说明

###4.1初始化SDK

 使用C-Life SDK，需要先调用sdk初始化方法，建议在Application中调用，需传入申请的AppID和AppSecret
 用户可以根据功能需要，在初始化的时候传入configModel，configModel可以设置是否开启wifi功能，蓝牙功能，设置请求地址环境，网络超时时间，是否打印log信息等，请参考model说明

	```
	/**
	 * 初始化SDK
	 * @param context 上下文常量
	 * @param appId    用户appid
	 * @param secret    用户密匙
	 * @param configModel    初始配置
	 */
	 public void init(Context context, String appId,
	                 String secret,ConfigModel configModel){
	```
###4.2 授权登录

   使用auth登录的方式登录clife开放平台，只有登录后，用户才可以绑定，控制，管理设备
   
   4.2.1 授权登录界面如下图
       授权登录，用户输入账号密码登录
       若用户没有clife账号，需要先注册clife账号
       若用户忘记密码，可以使用找回密码功能找回密码

   ![](http://i.imgur.com/bO48qSQ.jpg)
     
                 
###4.3 设备绑定
   > 目前，支持蓝牙设备，wifi设备绑定
   
   绑定的流程为，获取设备大类，获取设备小类，输入当前wifi密码，开始绑定，获取绑定结果回调
   
   
   4.3.1 获取绑定设备大类列表api
   
	 ```
	 HetDeviceListApi.getInstance().getTypeList(new ICallback<String>() {
	 ```
     
   4.3.2 获取绑定设备小类列表api
   
	 ```
	 HetDeviceListApi.getInstance().getSubTypeListProduct {  
	 ```
   
   4.3.3 WIFI设备绑定
   
   wifi绑定设备调用HetWifiBindApi，接口回调iWifiBind
     
	```
	/**
	 * 开始绑定设备
	 *
	 * @param wifiPassword wifi密码
	 * @param deviceModel  绑定的设备类型
	 * @param iWifiBind    绑定接口回调
	 */
	public void startBind(String wifiPassword, DeviceModel deviceModel, IWifiBind iWifiBind) {
	```
	
   4.3.4 蓝牙设备绑定 
   
   蓝牙绑定api使用HetBleBindApi: 
   [1] 使用前需调用初始化蓝牙api
	  
	```
	  /**
	  初始化蓝牙api
	  @return 0 成功 1 未开启蓝牙 2该手机不支持蓝牙功能
	  */
	  public int init() {
	```
		  
   [2] 蓝牙绑定api，回调接口IBleBind
		
	```
		/**
		* 开始绑定设备
		*
		* @param deviceModel 绑定的设备类型
		* @param iBleBind    绑定接口回调
		*/
		public void startBind(DeviceModel deviceModel, IBleBind iBleBind) {
	```

   [3] 连接蓝牙设备,由于蓝牙可能同时扫描到多个设备，用户需选择指定的蓝牙设备 

	```
		/**
		* 开始绑定设备
		*
		* @param deviceModel 绑定的设备类型
		* @param iBleBind    绑定接口回调
		*/
		public void startBind(DeviceModel deviceModel, IBleBind iBleBind) {
	```
			
   4.3.5 直接绑定
     
   开发者知道设备的类型（参考devicemodel字段说明），可以直接调用绑定方法，
   无需先选择大类，小类
      
##5设备管理
    
###5.1获取绑定设备列表
   
   绑定成功后，用户可以获取绑定成功的设备列表，获取到设备列表，才可以控制设备

  ```
  HetDeviceListApi.getInstance().getBindList(new IDeviceList() {
  ```
###5.2修改设备信息

   修改设备信息，用户可以修改设备的名称，使用HetDeviceManagerApi
        
	```
	/**
	* 更新设备信息
	*
	* @param deviceModel  设备model
	* @param iHetCallback 回调
	* @param deviceName   设备名称
	*/
	public void update(DeviceModel deviceModel, final IHetCallback iHetCallback, 
	String deviceName) {   
	```
    
###5.3解绑设备
    
   接触设备和clife平台的绑定关系，使用HetDeviceManagerApi
        
	```
		/**
		* 解绑设备
		*
		* @param deviceModel
		* @param iHetCallback
		*/
		public void unBind(DeviceModel deviceModel, final IHetCallback iHetCallback) {
	
	```
	   
###5.4获取设备协议
   
   获取设备在clife平台编辑的协议，在demo里面，进行设备控制的时候也会先获取设备的协议使用HetProtocolApi

 	```
     /***
     * 获取协议
     * @param iHetCallback  回调
     * @param productId 产品id
     * @param type 协议类型 0或者不传-完整协议	，包括以下协议内容 1-设备基本信息 
     * 2-控制数据 3-运行数据 4-故障数据
     */
   	  public  void getProtocol(final IHetCallback iHetCallback,int productId,int type){
	   	   ```
    
###5.5 获取设备历史数据
 
   获取设备历史数据，分为运行数据，控制数据，异常数据，最多获取7天的历史数据使用HetDeviceDataHistroyApi
   
   [1] 运行数据

   ```
     /**
     * 获取设备运行数据列表（七天之内）
     *
     * @param iHetCallback
     * @param deviceId	是	string	设备标识
     * @param startDate	是	Date	开始时间
     * @param endDate	否	Date	结束时间（默认为当天）
     * @param pageRows	否	int	每页显示的行数，默认为20
     * @param pageIndex	否	int	当前页，默认为1
     */
    public void getRunDataList(final IHetCallback iHetCallback, String deviceId, String startDate,String  endDate,int pageRows, int pageIndex) {
   ```
   	
   [2] 控制数据

   ```
		/**
		* 获取设备控制数据列表（七天之内）
		*
		* @param iHetCallback
		* @param deviceId	是	string	设备标识
		* @param startDate	是	Date	开始时间
		* @param endDate	否	Date	结束时间（默认为当天）
		* @param pageRows	否	int	每页显示的行数，默认为20
		* @param pageIndex	否	int	当前页，默认为1
		*/
		public void getConfigDataList(final IHetCallback iHetCallback,String deviceId, String startDate,String  endDate,int pageRows, int pageIndex) {
   ```
   
   [3] 异常数据

  ```
     /**
     * 获取设备故障数据列表（七天之内）
     *
     * @param iHetCallback
     * @param deviceId	是	string	设备标识
     * @param startDate	是	Date	开始时间
     * @param endDate	否	Date	结束时间（默认为当天）
     * @param pageRows	否	int	每页显示的行数，默认为20
     * @param pageIndex	否	int	当前页，默认为1
     */
    public void getErrorDataList(final IHetCallback iHetCallback, String deviceId, String startDate,String  endDate,int pageRows, int pageIndex) {
 	```
 	
	
##6 WIFI设备控制

   用户需要根据在clife平台上注册的设备协议，定义model,后面的sdk版本会自动从后台拉取协议解析。
目前demo 里定义了香薰机的model，请参考定义需要使用的model
  
###6.1 获取数据
   
   获取数据，指获取设备的运行数据和配置数据，使用HetDeviceSubmitApi方法
     
   [1] 初始化，数据回调接口 IDeviceUpdateView 
     
  ```
      /**
      * 初始化设备请求数据
      * @param deviceModel  设备model(必须获取deviceid后)
      * @param deviceUpdateView  获取数据接口回调
      */
      public void init(DeviceModel deviceModel,IDeviceUpdateView deviceUpdateView) {
	```
	 
   若用户未配置协议，需继承小循环model解析接口 IUdpModelParser，并在初始化的时候传递，用户需根据设备类型解析字节数据  
     
   ```
		/**
		* 初始化设备请求数据
		* @param deviceModel  设备model
		* @param parse        小循环数据解析
		* @param deviceUpdateView  获取数据接口回调
		*/
		public void init(DeviceModel deviceModel, IUdpModelParser parse, IDeviceUpdateView deviceUpdateView) {
   ```
		
   [2] 开始刷新数据，指开始获取设备的数据，默认每5s请求一次，用户可以设置时间间隔
     
	```
		/**
		* 开始请求数据
		*/
		public void start() {
	```
		
   [3] 停止请求数据
	   
  ```
	/**
	* 停止请求数据
	*/
	public void stop() {
  ```
  
   [4] 释放资源
  
	```
		/**
		* 释放请求资源
		*/
		public void destory() {
	```

###6.2 下发数据
   
   下发数据指，发送相关命令操作设备，发送调用HetDeviceSubmitApi方法
     
   [1] 关于updateflag
     
   这个修改标记位是为了做统计和配置下发的时候设备执行相应的功能。下发数据必须传递updateflag标志
  
  ```
    例如，空气净化器（广磊K180）配置信息协议：
	
    紫外线(1)、负离子(2)、臭氧(3)、儿童锁(4)、开关(5)、WiFi(6)、过滤网(7)、模式(8)、定时(9)、风量(10)
    上面一共上10个功能，那么updateFlag就2个字节，没超过8个功能为1个字节，超过8个为2个字节，超过16个为3个字节，以此类推。
	
    打开负离子，2个字节，每一个bit的值为下：
	
    0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
	```
    
   [2] 初始化
     
	```
	/**
	* 初始化
	* @param deviceModel  设备model
	* @param submitUpdateView  获取数据接口回调
	*/
	public void init(DeviceModel deviceModel,ISubmitUpdateView submitUpdateView) {
	```
	
   若用户未配置协议，需继承小循环model解析接口 IUdpModelParser，并在初始化的时候传递，用户需根据设备类型解析字节数据
     
	```
	/**
	* 初始化
	* @param deviceModel  设备model
	* @param submitUpdateView  获取数据接口回调
	*/
	public void init(DeviceModel deviceModel,ISubmitUpdateView submitUpdateView) {
	```
	
   [3] 开启下发数据

	```
	/**
	* 开启下发线程
	*/
	public void start() {
	```
	
   [4] 停止下发数据
		
	```
	/**
	* 停止下发数据
	*/
	public void stop() {
	```
		
   [5] 释放资源
     
	``` 
	/**
	* 释放相关资源
	*/
	public void destory() {
	```

###6.3直接控制wifi设备api

  指用户直接调用相关api请求clife平台控制wifi设备，使用HetDeviceWifiControlApi

  [1] 向设备下发控制数据
   
    /**
     * 向设备下发控制数据
     *
     * @param iHetCallback 调用成功的回调监听
     * @param deviceId 设备ID
     * @param json     设备控制数据
     */
    public static void setDataToDevice(final IHetCallback iHetCallback, String deviceId, String json) {

[2] 获取设备运行数据

      /**
     * 获取设备运行数据
     *
     * @param iHetCallback 调用成功的回调监听
     * @param deviceId 设备ID

     */
    public static void getRunFromDevice(final IHetCallback iHetCallback, String deviceId) {

[3]  获取设备控制数据

     /**
     * 获取设备控制数据
     *
     * @param iHetCallback 调用成功的回调监听
     * @param deviceId 设备ID

     */
    public static void getConfigFromDevice(final IHetCallback iHetCallback, String deviceId) {
[4]  获取设备异常数据

    /**
     * 获取设备异常数据
     *
     * @param iHetCallback 调用成功的回调监听
     * @param deviceId 设备ID

     */
    public static void getErrorDataFromDevice(final IHetCallback iHetCallback, String deviceId) {
   

    

	
##7蓝牙设备接入
   
   >第三方蓝牙设备接入指用户已开发好设备的的控制等功能，将设备接入clife开放平台，上传数据到clife平台，从clife平台获取  数据，蓝牙接入调用HetThridBleApi，需授权登录成功后才能使用相关api
   
###7.1提交设备信息到服务器
   
   将设备信息提交到服务器，设备信息可以通过大小分类获取，也可以默认传入，DEMO里面常用直接传入的形式 
   提交成功后，服务器会为该设备返回deviceId，只有获取deviceId后才能上传数据和获取历史数据
   
  ```
	/**
	* 将设备信息提交到和而泰服务器
	* @param deviceModel  设备model，
	* @param iHetCallback  回调
	*/
	public void bindToHetServer(final DeviceModel deviceModel, final IHetCallback iHetCallback) {
  ```
		
###7.2上传数据
	 以字节形式上传蓝牙设备的数据，设备数据分为实时数据和历史数据	
	 
	```
	/**
	* 上传数据
	*
	* @param iHetCallback
	* @param deviceModel  deviceModel
	* @param datas        数据
	* @param type         设备类型 1历史数据 2实时数据
	*/
	public void updateData(final IHetCallback iHetCallback,byte[] datas, DeviceModel deviceModel, int type) {
	```
	
   以json形式上传蓝牙设备的数据，设备数据分为实时数据和历史数据
   
	```
	/**
	* 上传数据
	*
	* @param iHetCallback
	* @param deviceModel  deviceModel
	* @param json        数据
	* @param type         设备类型 1历史数据 2实时数据
	*/
	public void updateData(final IHetCallback iHetCallback, String json,DeviceModel deviceModel, int type) {
	```
  
###7.3获取历史数据
   上传数据后，可以获取历史数据，数据以byte形式返回，详细的字段说明请参考 ThirdBLeDataModel
 
	```
	/**
	* 获取历史数据
	*
	* @param iHetCallback
	* @param deviceModel  deviceModel
	* @param order        排序方式（0-降序 1-升序 默认0-降序）
	* @param pageRows     每页显示的行数(默认20)
	* @param pageIndex    当前页（默认1）
	*/
	public void getData(final IHetCallback iHetCallback,DeviceModel deviceModel, int order,int pageRows, int pageIndex) {
	```

##8异常说明
   
###8.1全局返回码说明
  
   | 返回码 | 说明  |
   | --- | --- |
   | 0 | 请求成功 |
   | 100010100 | 缺少授权信息 |
   |100010101  |AccessToken错误或已过期|
   |100010102|RefreshToken错误或已过期|
   |100010103|AppId不合法|
   |100010104|timestamp过期|
   |100010105|签名错误|
   |100010106|请求地址错误|
   |100010107|请求Scheme错误|
   |100010108|请求方法错误|
   |100010109|服务处理超时|
   |100010110|用户未登录|
   |100010200|失败|
   |100010201|缺少参数|
   |100010202|参数错误|
   |100010203|必须使用https|
   |100010204|参数删除不允许|
   |100010205|推荐位置相同|
   |100021000|帐号已注册|
   |100021001|帐号未注册|
   |100021002|帐号错误|
   |100021003|帐号未激活|
   |100021004|帐号未邀请|
   |100021005|帐号邀请状态|
   |100021006|帐号在其它手机已登录|
   |100021100|手机号码已注册|
   |100021101|手机号码未注册|
   |100021200|邮箱已注册|
   |100021201|邮箱未注册|
   |100021300|发送验证码失败|
   |100021301|验证码错误|
   |100021302|随机码错误|
   |100021400|用户已存在|
   |100021401|用户不存在|
   |100021500|密码错误|
   |100021603|数据不存在|
   |100022000|设备不存在|
   |100022001|设备未绑定|
   |100022002|设备已绑定|
   |100022003|设备解绑失败|
   |100022004|MAC地址已绑定另一种设备|
   |100022005|设备控制JSON错误|
   |100022006|设备不在线|
   |100022800|命令发送失败|
   |100022008|不能邀请自己控制|
   |100022009|关联的设备不存在|
   |100022010|关联的设备未绑定|
   |100022011|设备已授权|
    
###8.2设备无法绑定
    
	   - 设备是否置为绑定模式
	
	   - 是否正确输入wifi密码
	
	   - 设备是否已在CLife开放平台注册
	
	   - APP端服务是否开启

 
















     

  







 
 
  


     
     
     



