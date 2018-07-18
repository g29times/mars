---
title: (转)Jpa CascadeType 详解
date: 2018-07-18 11:27:02
tags:
---
## Background
网上关于JPA的CascadeType讲解很多，但几乎都说的很模糊。
本文试图使用一个具体的例子来说明
CascadeType.PERSIST，CascadeType.MERGE，CascadeType.REFRESH，
CascadeType.REMOVE，CascadeType.ALL 具体区别。

首先，我们使用一个订单和订单项的例子。该例子在网络上那些介绍JPA CascadeType用法的文章钟广为流传。

```java
/**
 * 订单
 */
@Entity
@Table(name="t_order")
public class Order {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	@Column
	private String name;
	@OneToMany(mappedBy="order", targetEntity=Item.class, fetch=FetchType.LAZY)
	private List<Item> items;
}

/**
 *订单项
 */
@Entity
@Table(name="t_item")
public class Item {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	@Column
	private String name;
	@ManyToOne(fetch=FetchType.LAZY, targetEntity=Order.class)
	@JoinColumn(name="order_id")
	private Order order;
}

/** Order Repository */
public interface OrderRepository extends JpaRepository<Order, Integer>,
	JpaSpecificationExecutor<Order> {

}

/** Item Repository */
public interface ItemRepository extends JpaRepository<Item, Integer>,
	JpaSpecificationExecutor<Item> {

}
```


## 场景1，新增/(保存)数据(CascadeType.PERSIST)
客户每次下完订单后，需要保存Order，但是订单里含有Item。因此，在保存Order的时候，
Order相关联的Item也需要保存。采用上面的模型，使用如下的测试代码：

```java
@Test
public void addTest() {
	Order order = new Order();
	order.setName("order1");
	
	Item item1 = new Item();
	item1.setName("item1_order1");
	item1.setOrder(order);
	
	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);
	
	List<Item> items = new ArrayList<Item>();
	items.add(item1);
	items.add(item2);
	order.setItems(items);
	
	orderRepository.save(order);	
	Assert.assertEquals(1, orderRepository.count());
	Assert.assertEquals(2, itemRepository.count());
	
	//代码段1
	itemRepository.save(order);	
	Assert.assertEquals(1, orderRepository.count());
	Assert.assertEquals(2, itemRepository.count());
	
	//代码段2
	//itemRepository.save(items);
	//Assert.assertEquals(1, orderRepository.count());
	//Assert.assertEquals(2, itemRepository.count());
}
```
在该场景中，我们分别测试如下情况：

1. 没有任何CascadeType设置，由测试结果可知，order可以被保存到数据库，但是两个Item却不能。
2. 使用CascadeType.PERSIST
    1. 单独在Order类的items属性上加入cascade={CascadeType.PERSIST}，
    使用代码段1，order和items都可以被保存到数据库;
    使用代码段2，order和items都不能被保存到数据库。
    2. 单独在Item类的order属性上加入cascade={CascadeType.PERSIST}，
    使用代码段1，order可以被保存到数据库，items不可以被保存到数据库;
    使用代码段2，order和items都可以被保存到数据库。
    3. 在Order和Item类中都使用cascade={CascadeType.PERSIST}，
    使用代码段1，order和items都可以被保存到数据库;
    使用代码段2，order和items都可以被保存到数据库。
    由此可以知道，在某个类的属性上使用cascade={CascadeType.PERSIST}，对该类进行保存操作时，
    可以级连保存该类中此属性所对应的对象；
    而对该类的属性对应的对象进行保存操作时，却不能保存该类（存在外键时，二者都不可保存）。
    但是如果在该类和该类的属性所对应的类别中同时使用cascade={CascadeType.PERSIST}，
    那么无论是从该类出发进行保存操作，
    还是从该类的属性对应的对象出发进行保存操作，都可以保存二者。


## 场景2，删除数据(CascadeType.REMOVE)
现在有这样的场景，客户需要删除一个订单，那么订单中的订单项也需要一并删除，
为了可以实现级连删除的效果，我们使用以下测试代码：

```java
private Order order;
private List<Item> items = new ArrayList<Item>();

@Before
public void setUp() {
	order = new Order();
	order.setName("order1");
	Item item1 = new Item();
	
	item1.setName("item1_order1");
	item1.setOrder(order);
	
	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);
	
	items.add(item1);
	items.add(item2);
	order.setItems(items);
	
	orderRepository.save(order);
	itemRepository.save(items);
}

@Test
public void testDelete() {
	//代码段3
	orderRepository.delete(order);
	Assert.assertEquals(0, orderRepository.count());
	Assert.assertEquals(0, orderRepository.count());
	
	//代码段4
	itemRepository.delete(items);
	Assert.assertEquals(0, orderRepository.count());
	Assert.assertEquals(0, itemRepository.count());
}
```
在该场景中，我们分别测试如下情况：

1. 在Order和Item中都没有使用CascadeType.REMOVE时，单独删除order对象是不成功的，
这是由于数据库在item表中有基于order的外键约束。单独删除items可以成功的。
2. 使用CascadeType.REMOVE
    1. 在Order类的items属性上使用CascadeType.REMOVE，
    使用代码段3，通过Order对象的删除操作，可以级连删除order中items的Item对象
    (在删除过程中，会先删除items，然后再删除order)；
    但使用代码段4，虽然items可以成功删除，但是其关联的order对象却不能级连删除。
    2. 在Item类的order属性上使用CascadeType.REMOVE，
    使用代码段3，Order对象的删除操作是失败的，这是因为在存储item的表中有引用order表的外键；
    但是使用代码段4却可以成功的删除items及其级连的order对象。
    其过程是先更新items中引用的order的外键，设置items对order的引用为空值。
    然后删除items，然后再删除order。
    (注意，如果items为多条，那么会先删除一条item，然后删除order，然后再删除其余的item)。
    3. 在Order和Item中都使用CascadeType.REMOVE时，使用代码段3和代码段4都可以成功，
    这表明，在二者都使用CascadeType.REMOVE时，既可以通过删除order，同时级连删除items；
    也可以通过删除items，同时级连删除order。
通过以上的分析，可以了解到，在一般的业务场景中，需求基本是在删除order时同时级连删除items，
但反过来，在删除items的时候同时也要求删除order却不一定适合业务场景。
即使删除了所有和order相关的items，可能也需要保持住那个没有items的order。
所以这里的建议是，最好不要在order和item双方中都同时使用CascadeType.REMOVE，
即最好不要在关系的维护端(这里指Item类，因为item表中会有order的外键，
所以item是关系的维护端)使用CascadeType.REMOVE。


## 场景3，更新数据(CascadeType.MERGE)
在业务上，经常会有这样一种类似的需要：查找到了一个业务实体后，要更新该实体，
同时也需要更新该实体所关联的其他业务实体。在我们的例子中就是，同时需要更新Order和其所关联的Item。
我们使用如下测试代码：

```java
private Order order;
private List<Item> items = new ArrayList<Item>();

@Before
public void setUp(){
	order = new Order();
	order.setName("order1");
	Item item1 = new Item();
	
	item1.setName("item1_order1");
	item1.setOrder(order);
	
	Item item2 = new Item();
	item2.setName("item2_order1");
	item2.setOrder(order);
	
	items.add(item1);
	items.add(item2);
	order.setItems(items);
	
	orderRepository.save(order);
	itemRepository.save(items);
}

@Test
public void testUpdate(){
	order.setName("order1_updated");
	
	items.get(0).setName("item1_order1_updated");
	items.get(1).setName("item2_order1_updated");
	
	//代码段5
	orderRepository.save(order);
	Assert.assertEquals(1, orderRepository.count(new Specification<Order>(){
		public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "order1_updated");
		}
	}));		
	Assert.assertEquals(1, itemRepository.count(new Specification<Item>() {

		public Predicate toPredicate(Root<Item> root，CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "item1_order1_updated");
		}
	}));
	
	//代码段6
	itemRepository.save(items);
	Assert.assertEquals(1, itemRepository.count(new Specification<Item>() {
		public Predicate toPredicate(Root<Item> root，CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "item1_order1_updated");
		}
	}));
	Assert.assertEquals(1, orderRepository.count(new Specification<Order>(){
		public Predicate toPredicate(Root<Order> root, CriteriaQuery<?> cq, CriteriaBuilder cb) {
			return cb.equal(root.get("name").as(String.class), "order1_updated");
		}
	}));		
}
```
在该场景中，我们分别测试如下情况：

1. 在Order和Item中都没有使用CascadeType.MERGE时，使用代码段5，其测试结果表明更新Order成功，
但是并没有级连更新items；使用代码段6，更新items成功，但并没有级连更新items所关联的order对象。
2. 使用CascadeType.MERGE
    1. 单独在Order的items属性上使用CascadeType.MERGE，
    使用代码段5，其测试结果表明更新order成功，并且级连更新items也成功；
    使用代码段6，其测试结果表明，更新items成功，但是并没有级连更新order。
    2. 单独在Item的属性order上使用CascadeType.MERGE，
    使用代码段5，其测试结果表明更新items时，可以级连更新其关联的order对象；
    使用代码段6，更新items成功，但是并没有级连更新items关联的order对象。
    3. 在Order和Item中都使用CascadeType.MERGE时，使用代码段4和代码段5都可以成功，
    这表明，在二者都使用CascadeType.MERGE时，既可以通过更新order，同时级连更新items；
    也可以通过更新items，同时更新order。
通过以上的分析，可以了解到，通过使用CascadeType.MERGE，
可以通过更新关系的一端对象，而同时更新关系令一端的数据。


## 场景4，刷新数据(CascadeType.REFRESH)
这里刷新数据，是对应在这样的业务场景下：对于业务系统，一半会存在多个用户，
如果用户A取得了order和其对应的items，并且对order和items进行了修改，同时用户B也做了如此操作，
但是用户B先保存了，然后用户A保存时，需要先刷新order关联的items，然后再把用户A的变更更新到数据库。
这中场景就对应了CascadeType.REFRESH的需求。

[阅读原文](http://westerly-lzh.github.io/cn/2014/12/JPA-CascadeType-Explaining/)