---
layout: post
title:  ParamBean的设计
category: tech
---

删除服务里由于采用Mybatis封装了查询方法，经常会有Map<String,Object>类型的参数传入。目前主要有insert(插入删除记录),update(删除邮件撤回)这两个业务。若不对参数进行封装，则在业务代码中会初见较多的put(key,value)的代码，不易阅读且且不利于维护和排查错误，因此对ParamBean设计了基类。代码如下：

**BaseParamBean**
```java
public class BaseParamBean {

	private static final Logger LOG = LoggerFactory.getLogger(BaseParamBean.class);

	/**
   * 获取paramBean中的非空字段，作为HttpRequest参数
   *
   * @return
   * @throws ServiceException
   */
  public List<NameValuePair> getParams() throws ServiceException {
      List<NameValuePair> params = new ArrayList<NameValuePair>();
      Field[] field = this.getClass().getDeclaredFields();
      for (int j = 0; j < field.length; j++) {
          String fieldName = field[j].getName();
          String getterMethodName = getMethodName("get", fieldName);
          Method getterMethod = getMethod(getterMethodName);
          try {
              Object val = getterMethod.invoke(this);
              if (null != val) {
                  // 需要对数组类型的参数展开
                  if (val.getClass().isArray()) {
                      Object[] objects = (Object[]) val;
                      for (Object o: objects) {
                          NameValuePair pair = new BasicNameValuePair(fieldName, o.toString());
                          params.add(pair);
                      }
                  } else {
                      NameValuePair pair = new BasicNameValuePair(fieldName, val.toString());
                      params.add(pair);
                  }
              }
          } catch (Exception e) {
              LOG.error("parabean method invoke error, methodName:{}", getterMethodName, e);
              throw new ServiceException("PARABEAN.INVOKE.ERR", e);
          }
      }
      return params;
  }

  public static void main(String[] args) {
      Long[] xLongs = { 1L };
      System.out.println(xLongs.getClass().isArray());
  }

  /**
   * 获取paramBean中的非 空字段，作为Mybatis查询参数
   *
   * @return
   * @throws ServiceException
   */
  public Map<String, Object> getParamMap() throws ServiceException {
      Map<String, Object> params = new HashMap<String, Object>();
      Field[] field = this.getClass().getDeclaredFields();
      for (int j = 0; j < field.length; j++) {
          String fieldName = field[j].getName();
          String getterMethodName = getMethodName("get", fieldName);
          Method getterMethod = getMethod(getterMethodName);

          try {
              Object val = getterMethod.invoke(this);
              if (null != val) {
                  params.put(fieldName, getterMethod.invoke(this));
              }
          } catch (Exception e) {
              LOG.error("parabean method invoke error, methodName:{}", getterMethodName, e);
              throw new ServiceException("PARABEAN.INVOKE.ERR", e);
          }
      }
      return params;
  }

	/**
	 * 根据方法名获取paramBean中的方法
	 *
	 * @param methodName
	 * @return
	 * @throws ServiceException
	 */
	private Method getMethod(String methodName) throws ServiceException {
		try {
			return this.getClass().getMethod(methodName);
		} catch (SecurityException e) {
			LOG.error("security exception. methodName:{}", methodName, e);
			throw new ServiceException("PARABEAN.SECURITY.ERR", e);
		} catch (NoSuchMethodException e) {
			LOG.error("no such method in para bean. methodName:{}", methodName, e);
			throw new ServiceException("PARABEAN.NO.METHOD");
		}
	}

	/**
	 * 获取get/set方法名
	 *
	 * @param prefix
	 * @param filed
	 * @return
	 * @throws ServiceException
	 */
	private String getMethodName(String prefix, String filed) throws ServiceException {
		if (!prefix.equals("get") && !prefix.equals("set")) {
			LOG.error("method prefix must be get/set. prefix:{}, field:{}", prefix, filed);
			throw new ServiceException("PARABEAN.PREFIX.ERR");
		}
		return prefix + filed.substring(0, 1).toUpperCase() + filed.substring(1);
	}
}
```

**MailSearchBean**
```java
public class MailSearchBean extends BaseParamBean {

	private Long domainId;
	private String tid;
	private String mid;
	private String mailSubject;
	private Integer status;
	private Date startTime;
	private Date endTime;
	private String orderColumn;
	private String orderTurn = "DESC";

	//省略getters,setters

}

```

其中，子类的设计和普通的实体类相同，只是在父类中增加了对子类属性的查询方法。

参数类中的属性应当为封装类型，这样在反射调用get方法的过程中可以返回null而非默认值，从而对查询造成影响。

另外发现对第二个字母为大写的对象参数，java生成的get/set方法形如xString->getxString,这种情况下将首字母大写会造成找不到方法的问题，这种情况在hibernate中也存在，在项目中没有进行特殊处理，直接抛出异常。

之后再执行过程中，由于有些域需要传入数组类型的对象，这样使用Map类型的参数在mybatis中会造成冲突，因此在对参数进行解析的过程中，需要对参数的类型进行识别，数组类型的参数需要对其展开，循环添加至参数列表中。此外，在mybatis中的参数为Map类型，而HttpRequest查询中，参数类型为List<NameValuePair>。今天因为参数类型的修改，造成了这个问题，引以为戒，在设计通用方法的过程中也要注意。
