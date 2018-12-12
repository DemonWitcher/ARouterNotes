# ARouter

### 1.init 初始化
        LogisticsCenter判断
            debug模式或者当前是新版本就走下面的流程
                通过包名找出dex里的所有Arouter类
            不是debug模式并且当前版本有缓存
                就不分析dex了 直接加载缓存出来

        拿到所有文件后 按类名分类成interceptors,group（分组）,provider
        group会用class构造出对象,执行load函数,Warehouse类下的静态map里会存着所有分组信息，KEY是组名，value是IRouteGroup的class，
        里面存着这个分组下所有的activity和fragment路由信息

### 2.navigation 跳转
        build构造一个Postcard
        Postcard继承RouteMeta 是这里跳转的实体类
        navigation的真正实现在_ARouter的navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
        函数,274行
        LogisticsCenter.completion(postcard);
        尝试加载Warehouse类下静态map里保存的RouteMeta,没有的话,根据group去临时构造RouteMeta并且放到内存中
        完成后递归LogisticsCenter.completion再次尝试加载RouteMeta,成功后赋值路由信息给postcard
        下一步判断是否是URI跳转
        如果是fragment构造任务的话,会把postcard切换到绿色通道模式来跳过所有拦截器
        此时触发onFound回调
        判断是否是绿色通道跳转,不是的话就先运行拦截器，然后再执行_navigation完成跳转
        _navigation函数来执行具体跳转代码,构造intent,通过postcard赋值class信息
        通过new Handler（MainLooper）来完成跳转动作,有动画的再给动画,然后触发onArrival回调

        构造fragment的话 不经过拦截器 postcard自动切换到绿色通道
        通过postcard保存的class信息 直接构造fragment对象出来,赋值Arguments,然后返回实例

