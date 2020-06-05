---
description: 一级分类，二级分类，商品信息，购物车
---

# SSH综合案例-2

## 一级分类模块

在indexAction的excute方法中查询所有一级分类，查询后将集合存入到Session范围

```java
List<Category>  categoryList = categoryService.findAll();
// 存入到Session
ServletActionContext.getContext().getSession().put("categoryList", categoryList);
```

在jsp上通过\#session.categoryList获取数据

## 首页商品显示

在indexAction的excute方法中查询所有Product，显示前10个，分页查询

* 手动封装一个HibernateCallback的实现类

```java
public class PageHibernateCallback<T> implements HibernateCallback<List<T>>{

    private String hql;
    private Object[] params;
    private int startIndex;
    private int pageSize;


    public PageHibernateCallback(String hql, Object[] params,
            int startIndex, int pageSize) {
        super();
        this.hql = hql;
        this.params = params;
        this.startIndex = startIndex;
        this.pageSize = pageSize;
    }



    public List<T> doInHibernate(Session session) throws HibernateException,
            SQLException {
        //1 执行hql语句
        Query query = session.createQuery(hql);
        //2 实际参数
        if(params != null){
            for(int i = 0 ; i < params.length ; i ++){
                query.setParameter(i, params[i]);
            }
        }
        //3 分页
        query.setFirstResult(startIndex);
        query.setMaxResults(pageSize);

        return query.list();
    }

}
```

查询最新的十个商品

```java
// DAO层的查询最新商品
    public List<Product> findNew() {
        List<Product> list = this.getHibernateTemplate().executeFind(new PageHibernateCallback<Product>("from Product order by pdate desc", null , 0, 10));
        return list;
    }
```

在indexAction类中对newList提供set方法，在jsp页面上通过value="newList"来获取最新商品信息

## 分类页面

* 查询一级分类，以及一级分类关联二级分类，还有相应商品信息
* 配置一级分类和与二级分类的关联关系，二级分类与商品的关联关系
* 在一级分类的映射配置中，关闭延迟加载

```text
<set name="categorySeconds" order-by="csid" lazy="false">
```

* 循环嵌套显示

```text
<s:iterator value="categoryList" var="c">
            <dl>
                <dt><a href="${pageContext.request.contextPath}/product_findByCid.action?cid=<s:property value='%{#c.cid}'/>&page=1"><s:property value="#c.cname"/> </a></dt>
                <s:iterator value="#c.categorySeconds" var="cs">
                    <dd>
                        <a href="${pageContext.request.contextPath}/product_findByCsid.action?csid=<s:property value='%{#cs.csid}'/>&page=1"><s:property value="#cs.csname"/></a>
                    </dd>
                </s:iterator>
            </dl>
 </s:iterator>
```

* 在请求连接中带上分类的id，在Action中提供对应的set方法来获取，以便Action能获得到数据。
* 分别根据cid和csid查询商品，需要分页显示，封装数据到PageBean。

```java
public class PageBean<T> {
    private Integer page;// 当前页数.
    private Integer limit;// 每页显示记录数
    private Integer totalCount;// 总记录数
    private Integer totalPage;// 总页数.
    private List<T> list; // 显示到浏览器的数据.
//set/get方法。。。
}
```

* 在业务层将需要的数据封装到PageBean中，在Action中获得到PageBean数据

业务层

```java
public PageBean<Product> findByCsid(Integer csid, Integer page) {
        int limit =5;
        int totalPage=0;

        PageBean<Product> pageBean = new PageBean<Product>();

        pageBean.setPage(page);
        pageBean.setLimit(limit);


        //该分类下商品总数
        Integer totalCount = productDao.findCountByCsid(csid);

        pageBean.setTotalCount(totalCount);

        if(totalCount % limit == 0){
            totalPage = totalCount / limit;
        }else{
            totalPage = totalCount / limit + 1;
        }
        //总共页数
        pageBean.setTotalPage(totalPage);

        // 商品数据集合:
        int begin = (page - 1) * limit;
        List<Product> list = productDao.findByPageCsid(csid, begin ,limit);
        pageBean.setList(list);
        return pageBean;
    }
```

Dao

```java
    // 统计某个分类下的商品的总数:
    public Integer findCountByCid(Integer cid) {
        //String hql = "select count(*) from Product p , CategorySecond cs where p.categorySecond = cs and cs.category.cid = ?";
        String hql = "select count(*) from Product p join p.categorySecond cs join cs.category c where c.cid = ?";
        List<Long> list = this.getHibernateTemplate().find(hql,cid);
        System.out.println("list:============="+list.get(0).intValue());
        return list.get(0).intValue();
    }
        // 统计某个二级分类下商品数量
    public Integer findCountByCsid(Integer csid) {
        String hql = "select count(*) from Product p join p.categorySecond cs where cs.csid = ?";
        List<Long> list = this.getHibernateTemplate().find(hql,csid);
        return list.get(0).intValue();
    }
   //查某个一级分类下产品
    public List<Product> findByPage(Integer cid, int begin, int limit) {
        // String hql = "select p from Product p ,CategorySecond cs where p.categorySecond = cs and cs.category.cid = ?";
        String hql = "select p from Product p join p.categorySecond cs join cs.category c where c.cid = ?";
        List<Product> list = this.getHibernateTemplate().executeFind(new PageHibernateCallback<Product>(hql, new Object[]{cid}, begin, limit));
        return list;
    }
    //查某个二级分类下产品
    public List<Product> findByPageCsid(Integer csid, int begin, int limit) {
        String hql = "select p from Product p join p.categorySecond cs where cs.csid = ?";
        List<Product> list = this.getHibernateTemplate().executeFind(new PageHibernateCallback<Product>(hql, new Object[]{csid}, begin, limit));
        return list;
    }
```

* 在Action中将pageBean压栈，数据传递到页面

```java
ActionContext.getContext().getValueStack().set("pageBean",pageBean);
```

* 显示商品列表

```text
<ul>
    <s:iterator var="p" value="pageBean.list">
        <li>
            <a href="${pageContext.request.contextPath}/product_findByPid.action?pid=<s:property value="#p.pid"/>">
            <img src="${pageContext.request.contextPath}/<s:property value="#p.image"/>" width="170" height="170"  style="display: inline-block;">
            <span style='color:green'>
                <s:property value="#p.pname"/>
            </span>                     
            <span class="price">
                商城价： ￥<s:property value="#p.shop_price"/>
            </span>
            </a>
        </li>
    </s:iterator>    
</ul>
```

## 页码显示

根据一级商品和二级商品对应的action不同，所以提供两个不同的jsp页面

```text
<div class="pagination">
    第  <s:property value="pageBean.page"/>/<s:property value="pageBean.totalPage"/>页
    <s:if test="pageBean.page != 1">
        <a href="${ pageContext.request.contextPath }/product_findByCid.action?cid=<s:property value="cid"/>&page=1" class="firstPage">&nbsp;</a>        
        <a href="${ pageContext.request.contextPath }/product_findByCid.action?cid=<s:property value="cid"/>&page=<s:property value="pageBean.page-1"/>" class="previousPage">&nbsp;</a>    
    </s:if>    
    <s:iterator var="i" begin="1" end="pageBean.totalPage" step="1">
        <s:if test="pageBean.page==#i">
            <span class="currentPage"><s:property value="#i"/></span>
        </s:if>
        <s:else>
            <a href="${ pageContext.request.contextPath }/product_findByCid.action?cid=<s:property value="cid"/>&page=<s:property value="#i"/>"><s:property value="#i"/></a>
        </s:else>
    </s:iterator>            
    <s:if test="pageBean.page != pageBean.totalPage">
        <a class="nextPage" href="${ pageContext.request.contextPath }/product_findByCid.action?cid=<s:property value="cid"/>&page=<s:property value="pageBean.page+1"/>">&nbsp;</a>
        <a class="lastPage" href="${ pageContext.request.contextPath }/product_findByCid.action?cid=<s:property value="cid"/>&page=<s:property value="pageBean.totalPage"/>">&nbsp;</a>
    </s:if>    
</div>
```

## 显示商品的详细信息

在ProductAction中注入Product模型在jsp页面上通过model获取查询到的商品信息

```text
<div class="name"><s:property value="model.pname"/> </div>
        <div class="sn">
            <div>编号<s:property value="model.pid"/> </div>
        </div>
        <div class="info">
            <dl>
                <dt>商城价:</dt>
                <dd>
                    <strong>￥：<s:property value="model.shop_price"/> </strong>
                    参 考 价：
                    <del>￥<s:property value="model.market_price"/> </del>
                </dd>
            </dl>
        <div id="introduction" name="introduction" class="introduction">
            <div class="title">
                <strong><s:property value="model.pdesc"/> </strong>
            </div>
            <div>
                <img src="${ pageContext.request.contextPath}/<s:property value="model.image"/> ">
            </div>
        </div>
    </div>
</div>
```

## 购物车

### 购物项

把商品，数量和小计封装成购物项，其中小计应该有计算所得，不能有set方法

```java
ublic class CartItem {
    private Product product;

    private Integer count;
    private Double subtotal;

    public Product getProduct() {
        return product;
    }

    public void setProduct(Product product) {
        this.product = product;
    }

    public Integer getCount() {
        return count;
    }

    public void setCount(Integer count) {
        this.count = count;
    }

    public Double getSubtotal() {
        subtotal = count * product.getShop_price();
        return subtotal;
    }

}
```

### 购物车对象

购物车对象应具备的三个方法，添加、删除和清空

* map类型的商品列表，用pid作为key，购物项为value
* double类型商品总金额，只提供get方法
* 单独把购物项作为一个属性，方便在页面上访问

商品金额

```java
public class Cart {
    private Map<Integer,CartItem> map = new HashMap<Integer, CartItem>();
    private Double total=0d;
    private Collection cartItems;

    public Collection getCartItems(){
        return map.values();
    }
    public Double getTotal(){
        return total;
    }


    //add
    public void addCart(CartItem cartItem){
        Integer pid =cartItem.getProduct().getPid();
        if(map.containsKey(pid)){
            CartItem _cartItem = map.get(pid);
            _cartItem.setCount(_cartItem.getCount()+cartItem.getCount());
        }else {
            map.put(pid,cartItem);

        }
        total += cartItem.getSubtotal();
    }
    //delete
    public void removeCart(Integer pid){
        CartItem cartItem = map.remove(pid);
        total -= cartItem.getSubtotal();
    }
    //clear all
    public void clearCart(){
        map.clear();
        total = 0d;
    }

}
```

### CartAction

* pid属性和get方法，从jsp页面上获取到商品的pid
* ProductService注入，根据pid查找商品信息
* 从session中获得cart对象的方法request.getSession\(\).getAttribute\("cart"\)，第一次访问需要创建

```java
public class CartAction extends ActionSupport {

    private Integer pid;
    private Integer count;
    private ProductService productService;

    public void setPid(Integer pid) {
        this.pid = pid;
    }

    public void setCount(Integer count) {
        this.count = count;
    }

    public void setProductService(ProductService productService) {
        this.productService = productService;
    }

    public Cart getCart(HttpServletRequest request){
        Cart cart = (Cart) request.getSession().getAttribute("cart");
        if(cart ==null){
            cart = new Cart();
            request.getSession().setAttribute("cart",cart);
        }

        return cart;

...

    }
```

