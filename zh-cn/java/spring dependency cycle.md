Spring dependency cycle

在很多时候我们都会讨论Spring循环依赖的问题，Spring通过在框架层面来解决循环依赖，几乎不需要开发人员额外处理，但还是要遵循一些规范，比如：

1、循环依赖相关类的作用范围必须是Singleton的

2、必须使用Setter方式来注入依赖

接下来，我们先从逻辑上推理Spring如何解决循环依赖，然后再通过阅读源代码来验证我们的假设。

现在有三个类，代码如下：

```java
@Component
public class ServiceA{
    @Autowired
    private ServiceB serviceB;
}
```

```java
@Component
public class ServiceB{
    @Autowired
    private ServiceC serviceC;
}
```

```java
@Component
public class ServiceC{
    @Autowired
    private ServiceA serviceA;
}
```

从代码可以看出，ServiceA依赖ServiceB，ServiceB依赖ServiceC，最后ServiceC又依赖ServiceA，从而形成了循环依赖。我们假设Spring容器启动过程中，先加载ServiceA，那么整个加载和初始化类的过程如下：

1、Spring 首先尝试实例化ServiceA

2、接着填充ServiceA，发现ServiceA依赖ServiceB,而且在Spring容器中找不到ServiceB的实例

3、于是Spring尝试实例化ServiceB

4、接着填充ServiceB，发现ServiceB依赖ServiceC，而且在Spring容器中找不到ServiceC的实例

5、于是Spring尝试实例化ServiceC

6、接着填充ServiceC，发现ServiceC依赖ServiceA，而且在Spring容器中有一个完成实例化的ServiceA，于是用这个半成品的ServiceA实例来初始化ServiceC，这样ServiceC就完成了初始化

7、ServiceC完成初始化之后，从第4步返回，这样ServiceB完成填充ServiceC的实例，ServiceB完成初始化

8、ServiceB完成初始化之后，从第2步返回，这样ServiceA完成填充ServiceB的实例，ServiceA完成初始化

9、至此，ServiceA、ServiceB、ServiceC三个对象都完成了初始化

通过上面的步骤可以看出，一个完整的对象需要经历“实例化”和"初始化"两个阶段。在实例化阶段仅仅完成对象的创建，对象内部的依赖并未初始化，第二个阶段才开始初始化，在初始化完成之前，并不影响在Spring容器中通过beanName来访问这个对象(虽然这是不安全的)。我们可以通过缓存对象的引用，在它还未完成初始化之前提前将实例化的对象暴露出来，这样依赖该半成品对象完成初始化的对象就能顺利进行初始化过程，由于对象之间是通过引用的方式进行依赖的，提前暴露的半成品对象在完成初始化之后与我们缓存的成品对象都指向同一个对象，这样当所有对象的初始化动作都进行完成之后，对象就都是完整的了。

以上这个分析过程在网络上很多文章都能看到，那我们很好奇Spring到底是采用了什么方式来解决循环依赖的呢？接下来我们通过阅读源码的方式来详细分析Spring解决循环依赖的过程，本文以Spring 5.2.5-RELEASE版源码为例。

在分析源码之前，我先将Spring获取Bean实例的时序图放出来，以方便我们接下来的代码分析：

![](C:\Users\Lenovo\Downloads\spring dependency cycle.png)

一般来说，我们都是通过调用AbstractApplicationContext的getBean()方法来获取一个bean的实例，代码如下所示:

![image-20200721173758148](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200721173758148.png)

通过代码可知，getBean方法委托给对应的BeanFactory的getBean方法执行，代码如下所示：

![image-20200721173943107](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200721173943107.png)

在AbstractBeanFactory的getBean方法中，直接调用了该类的一个protected的方法doGetBean方法，我们先分析下这个方法，为了方便分析，我把与创建Bean不相关的代码给折叠起来，如下所示：

```java
//... 表示这中间的部分代码予以删减，但不影响我们整体理解创建bean的全过程

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		final String beanName = transformedBeanName(name);
		Object bean;
		//2.1.1 这里首先调用getSingleton(beanName)看看缓存中是否存在对象实例,当对象第一次创建时，这里的sharedInstance一定为null
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			//...
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
            //...
			try {
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);
				//...
				if (mbd.isSingleton()) {
                    // 2.1.2 调用getSingleton(beanName,ObjectFactory)方法来创建半成品对象实例
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							//...
                            destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					//...
				}
				else {
					//...
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}
		//...
		return (T) bean;
	}
```

上面的代码中重点关注注释为2.1.1和2.1.2的部分。主要包含两个步骤，先从对象实例缓存中判断对象是否已经存在，如果存在，直接就返回，如果不存在，则再次调用getSingleton的重载方法，去创建单例对象。下面我们先看下2.1.1部分的getSingleton方法，这个方法位于DefaultSingletonBeanRegistry类：

```java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

public Object getSingleton(String beanName) {
    return getSingleton(beanName, true);
}

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //它先从存放成品对象的缓存中判断是否存在该对象实例
    Object singletonObject = this.singletonObjects.get(beanName);
    //如果不存在，并且当前这个对象实例正在创建过程中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            //再从提前暴露对象的缓存中判断是否存在该对象实例
            singletonObject = this.earlySingletonObjects.get(beanName);
            // 如果对象还是不存在，同时允许提前暴露引用
            if (singletonObject == null && allowEarlyReference) {
                // 这时就可以通过singletonFactories中缓存的ObjectFactory来实例化这个对象，并把它放到earlySingletonObjects缓存中
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

上面的代码就是我们经常所说的三级缓存。当第一次创建实例时，这个方法一定返回NULL，因为singletonFactories返回的singletonFactory是空的。

对于这个方法我们有两个疑问，疑问1是isSingletonCurrentlyInCreation这个方法用于判断该对象是否处于创建中的状态，那么这个对象是什么时候被置为创建中的状态？疑问2是singletonFactories这个缓存中的对象是什么时候填充的呢？

接下来我们来看2.1.2部分getSingleton的重载方法，它传入了一个匿名类，这个重载方法同样位于DefaultSingletonBeanRegistry类，代码如下：

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				//...
				beforeSingletonCreation(beanName);
				//...
				try {
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
```

这个方法首先调用了beforeSingletonCreation方法，我们看看它是干什么的，代码如下所示：

```java
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}

```

可以看到它会将这个beanName放入到singletonsCurrentlyInCreation集合中，表示该beanName处于创建中的状态，这就回答了我们上面的疑问1。

我们接着看getSingleton的重载方法，接下来它调用了singletonFactory的getObject方法，其实就是调用我们在doGetBean方法中声明的那个匿名类的createBean方法，createBean位于AbstractAutowireCapableBeanFactory类中，如下所示：

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
		RootBeanDefinition mbdToUse = mbd;
		//...
		try {
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
		}
	}
```

这个方法内部调用了doCreateBean方法，如下文所示：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		//...
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			//...
            addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}
		//...
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}
```

在这里它先通过组合条件判断该Bean是否允许提前暴露，如果允许的话，调用addSingletonFactory方法，我们看下该方法的实现：

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

可以看出，该方法完成了singletonFactories的填充，这里也就弄清楚了我们前面提出的疑问2。

接下来，doCreateBean方法内部调用了populateBean这个方法，顾名思义，它就是用来填充bean的依赖属性的，接下来，我们重点看看它是如何填充Bean类型的属性，populateBean位于AbstractAutowireCapableBeanFactory类中，源码如下：

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		//...
		if (pvs != null) {
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

这个方法前面所有的预处理都可以跳过，重点看下applyPropertyValues这个方法，如下所示：

```java
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
		//...
		List<PropertyValue> deepCopy = new ArrayList<>(original.size());
		boolean resolveNecessary = false;
		for (PropertyValue pv : original) {
			if (pv.isConverted()) {
				deepCopy.add(pv);
			}
			else {
                //...这行代码才真正开始执行属性的填充
				Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
				//...
			}
		}
		//...
	}
```

主要看valueResolver.resolveValueIfNecessary(pv, originalValue)这行代码，这个方法位于BeanDefinitionValueResolver；通过这个方法的doc我们可以看出这个方法主要是根据给出的PropertyValue，返回一个容器中的对象引用，这个方法中有大量的逻辑分支，我们重点关注value instanceof BeanDefinition 这个分支,代码如下：

```java
public Object resolveValueIfNecessary(Object argName, @Nullable Object value) {
		//...
		else if (value instanceof BeanDefinition) {
			// Resolve plain BeanDefinition, without contained name: use dummy name.
			BeanDefinition bd = (BeanDefinition) value;
			String innerBeanName = "(inner bean)" + BeanFactoryUtils.GENERATED_BEAN_NAME_SEPARATOR +
					ObjectUtils.getIdentityHexString(bd);
			return resolveInnerBean(argName, innerBeanName, bd);
		}
		//...
	}
```

它调用了resolveInnerBean这个方法，如下所示：

```java
private Object resolveInnerBean(Object argName, String innerBeanName, BeanDefinition innerBd) {
		RootBeanDefinition mbd = null;
		try {
			//...
			Object innerBean = this.beanFactory.createBean(actualInnerBeanName, mbd, null);
			if (innerBean instanceof FactoryBean) {
				boolean synthetic = mbd.isSynthetic();
				innerBean = this.beanFactory.getObjectFromFactoryBean(
						(FactoryBean<?>) innerBean, actualInnerBeanName, !synthetic);
			}
			if (innerBean instanceof NullBean) {
				innerBean = null;
			}
			return innerBean;
		}
		catch (BeansException ex) {
			throw new BeanCreationException(
					this.beanDefinition.getResourceDescription(), this.beanName,
					"Cannot create inner bean '" + innerBeanName + "' " +
					(mbd != null && mbd.getBeanClassName() != null ? "of type [" + mbd.getBeanClassName() + "] " : "") +
					"while setting " + argName, ex);
		}
	}
```

在这里我们可以看到，如果填充的bean在容器中不存在时，它也会去调用beanFactory的createBean方法，这样又回到前面时序图中的2.1.2.1步骤，循环往复，直到完成所有bean的初始化。

到这里，我们从源码层面分析了Spring是如何来处理循环依赖问题的。总的来说，它采用了三级缓存来缓存半成品对象和成品对象，同时基于对象的创建过程分为实例化和初始化两个阶段，才得以解决循环依赖。

