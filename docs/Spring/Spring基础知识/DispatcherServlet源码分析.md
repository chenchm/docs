## 请求从DispatcherServlet到达具体的Controller

## DispatcherServlet

Servlet的service方法是响应前端请求的入口方法，Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。DispatcherServlet是一个标准的Servlet，肯定具有service方法，并且通过service方法来响应所有的请求。

DispatherServlet继承自FrameWorkServlet，FrameWorkServlet是 springmvc 的基础 Servlet，它的主要工作是对 `WebApplicationContext`实例进行初始化和管理，并且FrameWorkServlet重写了doGet、doPost、doPut，doDelete 等方法，将所有类型的请求都交给processRequest方法来处理。

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
        //抽象方法由子类实现
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

processRequest方法主要是作用是初始化request请求的上下文参数，然后将请求交给子类的doService方法处理，该方法由DispatcherServlet实现。

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    //保存request请求参数的快照，使在后续处理中可以恢复原来的数据
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // 添加框架的内置对象到request域中，使得handler和view对象能够使用
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    //注入通过redirect传递来的（RedirectAttribute）属性
    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    try {
        //进行请求分发
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

可以看出doService方法主要作用是对request请求参数进行备份，并且对request必要域进行赋值操作，最后调用doDispatch方法进行请求分发。

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            //判断当前请求是否是文件请求,如果是的话就将请求转换为文件请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            //找到能够处理当前请求的handler
            //该对象最终会被包装为HandlerExecutionChain类型
            mappedHandler = getHandler(processedRequest);
            
            ...
        }
    }
}
```

DispatcherServlet的getHandler方法根据请求获取请求所对应的mappedHandler（即具体处理请求的 method 方法，实际上会最终被包装成一个`HandlerExecutionChain` 对象），这里是找到响应该请求的Controller方法的关键，该方法的具体实现如下：

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        //遍历所有的HandlerMapping对象，调用其getHandler
        //如果该方法返回的handler不为空，则返回第一个获得的handler
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

以上的逻辑，只是遍历了DispatcherServlet的handlerMappings集合，然后调用HandlerMapping的getHandler方法获得HandlerExecutionChain对象（主要是包括了一个handler，以及一个拦截器列表），如果该对象不为空则返回该对象。这里问题就出现了，handlerMappings集合是什么，是从哪里获得的？

实际上handlerMappings集合中存放了实现了HandlerMapping接口的对象，其中最重要的对象就是`RequestMappingHandlerMapping`对象，它会在spring容器初始化以后将Controller中添加了@RequestMapping注解的方法映射信息缓存起来，方便在响应请求时通过反射调用对应的方法，具体实现将在下文进行分析。那么RequestMappingHandlerMapping是在哪配置的，又是怎么初始化的呢？直接提起RequestMappingHandlerMapping对象大家可能不太熟悉，但是spring配置文件中的`<mvc:annotation-driven/>`配置项大家应该不陌生，该配置项默认添加了以下配置：

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
```

因此在spring容器初始化时，会将RequestMappingHandlerMapping对象初始化，而该对象的父类AbstractHandlerMethodMapping实现了`InitializingBean`接口并实现其afterPropertiesSet方法，在Bean依赖注入完成时，spring会去检查这个Bean是否实现了InitializingBean接口，如果实现了InitializingBean接口，就会去调用这个类的afterPropertiesSet方法。AbstractHandlerMethodMapping的afterPropertiesSet方法如下：

```java
public void afterPropertiesSet() {
    initHandlerMethods();
}

protected void initHandlerMethods() {
    //对所有的BeanName调用processCandidateBean方法
    for (String beanName : getCandidateBeanNames()) {
        if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {
            processCandidateBean(beanName);
        }
    }
    handlerMethodsInitialized(getHandlerMethods());
}

//从spring上下文中获得所有的Bean的名字，因为这里的BeanType为Object
protected String[] getCandidateBeanNames() {
    return (this.detectHandlerMethodsInAncestorContexts ?
            BeanFactoryUtils.beanNamesForTypeIncludingAncestors(obtainApplicationContext(), Object.class) :
            obtainApplicationContext().getBeanNamesForType(Object.class));
}
```

afterPropertiesSet调用了initHandlerMethods方法获得当前spring上下文中所有bean的名字，并且对符合条件的beanName调用processCandidateBean方法。

```java
protected void processCandidateBean(String beanName) {
    Class<?> beanType = null;
    try {
        beanType = obtainApplicationContext().getType(beanName);
    }
    catch (Throwable ex) {
        // An unresolvable bean type, probably from a lazy bean - let's ignore it.
        if (logger.isTraceEnabled()) {
            logger.trace("Could not resolve type for bean '" + beanName + "'", ex);
        }
    }
    //beanType不为空并且isHandler方法返回值为true
    if (beanType != null && isHandler(beanType)) {
        detectHandlerMethods(beanName);
    }
}

//isHandler方法在RequestMappingHandlerMapping的实现
protected boolean isHandler(Class<?> beanType) {
    return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
            AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}
```

processCandidateBean方法通过beanName获得bean的类型，当bean的类型不为空且该类同时具有@Controller注解和@RequestMapping注解时，才会继续调用detectHandlerMethods方法。

```java
protected void detectHandlerMethods(Object handler) {
    	//根据beanName获得其class类型
		Class<?> handlerType = (handler instanceof String ?
				obtainApplicationContext().getType((String) handler) : handler.getClass());

		if (handlerType != null) {
			Class<?> userType = ClassUtils.getUserClass(handlerType);
             //遍历该类实现的所有方法，获取它的所有方法对应的匹配信息(@RequestMapping注解中的信息)
             //并且将匹配信息保存在mappingRegistry中
			Map<Method, T> methods = MethodIntrospector.selectMethods(userType,
					(MethodIntrospector.MetadataLookup<T>) method -> {
						try {
                               //根据方法匹配信息构造RequestMappingInfo
							return getMappingForMethod(method, userType);
						}
						catch (Throwable ex) {
							throw new IllegalStateException("Invalid mapping on handler class [" +
									userType.getName() + "]: " + method, ex);
						}
					});
			if (logger.isTraceEnabled()) {
				logger.trace(formatMappings(userType, methods));
			}
			methods.forEach((method, mapping) -> {
				Method invocableMethod = AopUtils.selectInvocableMethod(method, userType);
                 //将方法和@RequestMapping中的信息缓存到mappingRegister
				registerHandlerMethod(handler, invocableMethod, mapping);
			});
		}
	}

//getMappingForMethod方法在RequestMappingHandlerMapping的实现
protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
    //从方法上@RequestMapping注解中的属性封装到RequestMappingInfo中
    RequestMappingInfo info = createRequestMappingInfo(method);
    if (info != null) {
        //同样的从类中的上@RequestMapping注解中获得信息
        RequestMappingInfo typeInfo = createRequestMappingInfo(handlerType);
        if (typeInfo != null) {
            //将方法和类上的信息进行一次合并
            info = typeInfo.combine(info);
        }
        String prefix = getPathPrefix(handlerType);
        if (prefix != null) {
            info = RequestMappingInfo.paths(prefix).build().combine(info);
        }
    }
    return info;
}
```

detectHandlerMethods的作用主要是获得类所有的实现方法，遍历这些方法获得方法上@ReqestMapping中的信息，构造成RequestMappingInfo对象，需要注意的是在构造RequestMappingInfo时会将类上的注解信息和方法上的注解信息进行合并，比如类对应的path为 test,Method对应的path为 test1,那么最后合并的path即为 test/test1。最后，调用registerHandlerMethod将获得的信息缓存在mappingRegister中。

```java
protected void registerHandlerMethod(Object handler, Method method, T mapping) {
    //mappingRegistry是AbstractHandlerMethodMapping的内部类MappingRegistry的实体对象
    //主要缓存了MappingInfo与Method的映射，以及url与MappingInfo的映射
    this.mappingRegistry.register(mapping, handler, method);
}

public void register(T mapping, Object handler, Method method) {
    this.readWriteLock.writeLock().lock();
    try {
        //将handler(类名或类)和方法关联创建HandlerMethod对象
        HandlerMethod handlerMethod = createHandlerMethod(handler, method);
        assertUniqueMethodMapping(handlerMethod, mapping);
        //将MappingInfo与Method的映射存在mappingLookup中
        this.mappingLookup.put(mapping, handlerMethod);

        List<String> directUrls = getDirectUrls(mapping);
        for (String url : directUrls) {
            //将url与MappingInfo的映射存在urlLookup中
            this.urlLookup.add(url, mapping);
        }

        String name = null;
        if (getNamingStrategy() != null) {
            name = getNamingStrategy().getName(handlerMethod, mapping);
            addMappingName(name, handlerMethod);
        }

        CorsConfiguration corsConfig = initCorsConfiguration(handler, method, mapping);
        if (corsConfig != null) {
            this.corsLookup.put(handlerMethod, corsConfig);
        }

        this.registry.put(mapping, new MappingRegistration<>(mapping, handlerMethod, directUrls, name));
    }
    finally {
        this.readWriteLock.writeLock().unlock();
    }
}
```

到此RequestMappingHandlerMapping对象初始化基本完成，在DispatchServlet初始化时会通过BeanFactory获得所有实现了HandlerMapping的类来初始化handlerMappings成员变量。

现在让我们回到DispatcherServlet的getHandler方法上，该方法调用了HandlerMapping的getHandler方法，具体实现在AbstractHandlerMethodMapping中：

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    //获得handler对象
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = obtainApplicationContext().getBean(handlerName);
    }

	...
}

protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
    //从request中获得请求路径
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    //加锁
    this.mappingRegistry.acquireReadLock();
    try {
        //获得和请求路径匹配的HandlerMethod对象
        HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
        return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
    }
    finally {
        //释放锁
        this.mappingRegistry.releaseReadLock();
    }
}
```

AbstractHandlerMethodMapping的getHandler方法调用了getHandlerInternal来获得HandlerMethod对象，在getHandlerInternal方法中先mappingRegistry对象添加读锁，接着调用lookupHandlerMethod来获得具体的HandlerMethod，从之前的分析中我们知道了HandlerMethod由MappingRegister中进行了缓存，所以lookupHandlerMethod很可能是在缓存中寻找对应的HandlerMethod，方法如下：

```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
    List<Match> matches = new ArrayList<>();
    //根据url在mappingRegistry的urlLookup中找到直接路径匹配的MappingInfo
    List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);
    if (directPathMatches != null) { //如果url能够直接匹配
        //再根据具体信息找到最匹配的方法
        addMatchingMappings(directPathMatches, matches, request);
    }
    if (matches.isEmpty()) { //如果url不能直接匹配，这里考虑通配符的问题
        // No choice but to go through all mappings...
        addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);
    }

   ...
}
```

不出所料，在lookupHandlerMethod方法中先根据请求的url信息，在mappingRegistry的urlLookup中初步确定匹配的MappingInfo，然后调用addMatchingMappings方法中进行详细的筛选，找出匹配的MappingInfo；如果在直接路径匹配没有找到任何匹配的信息（由于通配符的原因），只能遍历mappingRegistry的mappingLookup中所有的MappingInfo来找到匹配的信息。addMatchingMappings方法如下：

```java
private void addMatchingMappings(Collection<T> mappings, List<Match> matches, HttpServletRequest request) {
    for (T mapping : mappings) {
        //进行详细的匹配判断
        T match = getMatchingMapping(mapping, request);
        if (match != null) {
            matches.add(new Match(match, this.mappingRegistry.getMappings().get(mapping)));
        }
    }
}

//在RequestMappingInfoHandlerMapping实现
protected RequestMappingInfo getMatchingMapping(RequestMappingInfo info, HttpServletRequest request) {
    return info.getMatchingCondition(request);
}

//最终调用了RequestMappingInfo的getMatchingCondition方法
public RequestMappingInfo getMatchingCondition(HttpServletRequest request) {
    RequestMethodsRequestCondition methods = this.methodsCondition.getMatchingCondition(request);
    ParamsRequestCondition params = this.paramsCondition.getMatchingCondition(request);
    HeadersRequestCondition headers = this.headersCondition.getMatchingCondition(request);
    ConsumesRequestCondition consumes = this.consumesCondition.getMatchingCondition(request);
    ProducesRequestCondition produces = this.producesCondition.getMatchingCondition(request);

    if (methods == null || params == null || headers == null || consumes == null || produces == null) {
        return null;
    }

    PatternsRequestCondition patterns = this.patternsCondition.getMatchingCondition(request);
    if (patterns == null) {
        return null;
    }

    RequestConditionHolder custom = this.customConditionHolder.getMatchingCondition(request);
    if (custom == null) {
        return null;
    }

    return new RequestMappingInfo(this.name, patterns,
                                  methods, params, headers, consumes, produces, custom.getCondition());
}
```

从上面的代码可以看出，在addMatchingMappings中调用getMatchingMapping方法对匹配信息进行精确判断，如果判断通过则加入到结果集合中。最终将调用到RequestMappingInfo的getMatchingCondition方法，该方法对7个要素进行了匹配判断：

- methods : 请求方法匹配（POST/GET/DELETE/PUT等等)，对应RequestMapping注解的method
- params : 请求参数匹配 例如myParam=myValue, 对应RequestMapping注解的params
- headers:请求头信息(例如：My-Header!=myValue），对应RequestMapping的headers
- consumes:提交内容类型(例如:application/json, text/html;),对应RequestMapping的consumes
- produces:返回的内容类型(例如:application/json),对应RequestMapping的consumes
- patterns:URL格式(例如:test/testDo),对应RequestMapping的value
- 自定义条件

全部要素都通过判断后新建一个RequestMappingInfo副本返回。

找到一个url匹配的所有方法后，会对匹配的方法进一步的比较，找出最匹配的方法（bestMatch）返回，lookupHandlerMethod方法后续代码如下：

```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
    List<Match> matches = new ArrayList<>();
   	...
        
    if (!matches.isEmpty()) { //如果找到的匹配方法数量大于0
        //通过定义的Comparator进行排序
        Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
        matches.sort(comparator);
        //第一个即是最匹配的方法
        Match bestMatch = matches.get(0);
        if (matches.size() > 1) {
            if (logger.isTraceEnabled()) {
                logger.trace(matches.size() + " matching mappings: " + matches);
            }
            if (CorsUtils.isPreFlightRequest(request)) {
                return PREFLIGHT_AMBIGUOUS_MATCH;
            }
            Match secondBestMatch = matches.get(1);
            if (comparator.compare(bestMatch, secondBestMatch) == 0) {
                Method m1 = bestMatch.handlerMethod.getMethod();
                Method m2 = secondBestMatch.handlerMethod.getMethod();
                String uri = request.getRequestURI();
                throw new IllegalStateException(
                    "Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
            }
        }
        request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.handlerMethod);
        handleMatch(bestMatch.mapping, lookupPath, request);
        return bestMatch.handlerMethod;
    }
    else { //否则进入没有匹配方法的逻辑
        return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
    }
}
```

找到最最匹配的方法后则返回HandlerMethod对象，让我们从AbstractHandlerMethodMapping的getHandler中往下走，后续代码如下：

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    Object handler = getHandlerInternal(request);
   	...

    //将HandlerMethod包装为HandlerExecutionChain
    //执行的操作就是将注册了对应URL的过滤器挂载到该chain中，形成一条方法执行链。
    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    ...
    //判断是否是cors请求，是的话添加对应的过滤器
    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.corsConfigurationSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }

    return executionChain;
}
```

包装完成后的handler将被回到DispatcherServlet的doDispatch方法中，继续往下执行：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    ...
    mappedHandler = getHandler(processedRequest);
    //如果没找到匹配的handler，调用noHandlerFound方法根据配置抛出异常或者返回404状态码
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
    }

    // 根据Handler获得对应的HandlerAdapter对象（即配置中的RequestMappingHandlerAdapter）
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    // 判断获得的handler是否支持last-modified请求头
    String method = request.getMethod();
    boolean isGet = "GET".equals(method);
    if (isGet || "HEAD".equals(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
            return;
        }
    }

    //执行请求过滤器的preHandler方法，返回TRUE时继续向下执行，返回FALSE代表请求被过滤了，这时直接返回
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
    }

    // 开始真正调用了handler逻辑
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

    if (asyncManager.isConcurrentHandlingStarted()) {
        return;
    }

    applyDefaultViewName(processedRequest, mv);
    mappedHandler.applyPostHandle(processedRequest, response, mv);
    ...
}
```

判断返回的handler的对象是否为空，若handler为空则抛出异常或者返回404；若handler不为空，根据handler获得对应的HandlerAdapter对象，HandlerAdapter是用来调用HandlerMethod对应的方法的适配器，在spring中具体为RequestMappingHandlerAdapter；之后会执行执行过滤链中的过滤方法，判断请求是否被应该过滤，完成过滤逻辑以后，才会调用HandlerAdapter的handle方法开始真正的方法调用。

```java
//在AbstractHandlerMethodAdapter的实现
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {

    return handleInternal(request, response, (HandlerMethod) handler);
}
```

在handle方法中将 HandlerExecutionChain强转回 HandlerMethod 对象，并传递给子类（RequestMappingHandlerAdapter）的 handlerInternal 方法来进行后续处理。

```java
protected ModelAndView handleInternal(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ModelAndView mav;
    // 判断请求类型是否被支持以及请求是否需要session
    checkRequest(request);

    // 判断invokeHandlerMethod方法是否需要在同步块中执行
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }

    if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
        if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
            applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
        }
        else {
            prepareResponse(response);
        }
    }

    return mav;
}
```

可以看出该方法的主要作用就是对当前请求进行一些判断与处理，然后调用最重要的方法invokeHandlerMethod，这里就不深入该方法的逻辑了（以后有时间再研究吧。。。），只对该方法进行简单的说明。在后续执行过程中会先对方法传入参数进行解析和匹配，如果没有匹配到合适的值就会抛出异常。一个典型的例子便是 `@RequestParam` 注解，如果你加了这个注解，那么参数名必须与表单名一致，否则会报错。之后会通过反射调用到具体的Controller方法，最后会对方法返回的结果进行处理，例如对添加了@ResponseBody注解的方法，会对其返回值进行json化并将其写入 *responseBody* 中。

## 参考

1.[http://ddrv.cn/a/58528](http://ddrv.cn/a/58528)

2.[https://www.jianshu.com/p/ef15e524458e](https://www.jianshu.com/p/ef15e524458e)