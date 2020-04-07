---
description: 后台一级分类，二级分类，商品的CURD 
---

# SSH综合案例-4

设置cascad = delete 级联删除的时候需要先查询一级，再删除，二级才能一起删除。注意级联删除的方向性



已经设置了模型注入，就不需要给同名属性再使用set方法来获取请求参数


自定义未登录拦截器

```java
public class LoginInterceptor extends MethodFilterInterceptor {

    @Override
    protected String doIntercept(ActionInvocation actionInvocation) throws Exception {

        AdminUser existAdminUser = (AdminUser) ServletActionContext.getRequest().getSession().getAttribute("existAdminUser");
        if(existAdminUser !=null){
            return  actionInvocation.invoke();

        }else {
            ActionSupport action = (ActionSupport) actionInvocation.getAction();
            action.addActionError("请先登录");
            return action.LOGIN;
        }
    }
}
```