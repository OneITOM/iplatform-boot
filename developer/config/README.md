# 集中配置开发手册

> 作者 于胜强

## 修改接口

> 通过RESTful API提供配置的新增、修改、删除

`POST` `https://localhost:8761/api/v1/configparams/{operation}?configName=XXX&configProfile=XXX&jsonParams=XXX`

> 参数说明：

- operation: `add`, `update`, `delete`
- configName：应用微服务ID 如：`dfss-service`
- configProfile：应用环境标签 如：`development`
- jsonParams：配置项json数组 如`[{"key":"testkey1","value":"testStr"},{"key":"testkey2","value":123456}]`

> 接口返回

- 正确返回：`{"succcess":true, "message":null, "data":null}`
- 错误返回：`{"succcess":false, "message":"您的参数不合法XXX.", "data":null}`

## 监听接口

> 订阅消息总线,实现配置变更后的逻辑

```java
@Service
@ConditionalOnExpression("${spring.cloud.config.enabled:false}")
public class ChangeMsgEventListener {
  private static final Logger LOG = LoggerFactory.getLogger(ChangeMsgEventListener.class);
  /** 装配变更通知消息总线 */
  @Autowired
  private AsyncEventBus asyncEventBus;
	
  @Value("${spring.application.name:}")
  private String application;
  @Value("${spring.cloud.config.profile:}")
  private String profile;    
  @Value("${spring.cloud.config.label:main}")
  private String label;
  @Value("${spring.cloud.config.busisys:OneITOM}")
  private String busiSys;
    
  /** 注册订阅方法至消息总线 */
  @PostConstruct
  public void register() {
    asyncEventBus.register(this);
  }
	
  /** 注册订阅变更通知事件 */
  @Subscribe
  public void listener(ChangeMsgEvent event) {
    //接收到的消息event
    LOG.info("###receive msg:"+event);
    //判断是否为自己的应用
    LOG.info(String.format("应用信息：busiSys:%s, application:%s, profile:%s, label:%s", busiSys, application, profile, label));
    if(busiSys.equals(event.getBusisys()) && application.equals(event.getApplication()) && 
      profile.equals(event.getProfile()) && label.equals(event.getLabel())) {

      //获取变更的配置
      List<PropItem> propList = event.getProps();

      //TODO:根据获取的变更配置逻辑处理

    } else {
      LOG.info("非自身应用变更通知消息，忽略.");
    }
  }
}
```
