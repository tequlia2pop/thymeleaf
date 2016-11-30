
Thymeleaf - Spring Security 集成模块
===============================================

------------------------------------------------------------------------------

**[请确保选择对应于您使用的Thymeleaf版本的分支]**

状态
------

这是*thymeaf extras*模块，不是Thymeleaf核心的一部分（以及自己的版本化模式），但由Thymeleaf团队提供完全的支持。

此仓库包含两个项目：

  * **thymeleaf-extras-springsecurity3** 用于与Spring Security 3.x集成
  * **thymeaf-extras-springsecurity4** 用于与Spring Security 4.x集成
 
当前版本：

  * **版本3.0.1.RELEASE** - 用于Thymeleaf 3.0（需要Thymeleaf 3.0.0+）
  * **版本2.1.3.RELEASE** - 用于Thymeleaf 2.1（需要Thymeleaf 2.1.2+）

License
-------

This software is licensed under the [Apache License 2.0]
(http://www.apache.org/licenses/LICENSE-2.0.html).

要求（3.0.x）
--------------------

  *   Thymeleaf **3.0.0+**
  *   Spring Framework 版本 **3.0.x** 到 **4.3.x**
  *   Spring Security 版本 **3.0.x** 到 **4.2.x**
  *   Web 环境 (Spring Security 集成无法离线工作)

Maven 信息
----------

  *   groupId: `org.thymeleaf.extras`   
  *   artifactId: 
    *   Spring Security 3 integration package: `thymeleaf-extras-springsecurity3`
    *   Spring Security 4 integration package: `thymeleaf-extras-springsecurity4`


Distribution packages
---------------------

Distribution packages (binaries + sources + javadoc) can be downloaded from [SourceForge](http://sourceforge.net/projects/thymeleaf/files/thymeleaf-extras-springsecurity/).


功能
--------

此模块提供了一个新的方言，称为`org.thymeleaf.extras.springsecurity3.dialect.SpringSecurityDialect`或`org.thymeleaf.extras.springsecurity4.dialect.SpringSecurityDialect`（取决于Spring Security版本），具有默认前缀`sec`。包括：
  
  *   新的表达式工具对象：
    *   `#authentication` 表示 Spring Security 认证对象（实现了 `org.springframework.security.core.Authentication` 接口的对象）。
	*   `#authorization`：表达式工具对象，它提供了一些方法，可以根据表达式、URL和访问控制列表检查授权。

  *   新的属性：
    *   `sec:authentication="prop"` 用于输出 authentication 对象的 `prop` 属性，类似于 Spring Security 的 `<sec:authentication/>` JSP 标签。
    *   `sec:authorize="expr"` 或 `sec:authorize-expr="expr"` 如果认证用户根据指定的 Spring Security 表达式被授权的话，它们将渲染 element children（*标签的内容*）。
    *   `sec:authorize-url="url"` 如果认证用户被授权查看指定的 URL，则渲染 element children（*标签的内容*）。
    *   `sec:authorize-acl="object :: permissions"` 根据 Spring Source 的访问控制列表（Access Control List）系统，如果认证用户对指定的领域对象具有指定的权限，则渲染 element children（*标签的内容*）。


------------------------------------------------------------------------------

	
配置
-------------

为了在我们的 Spring MVC 应用程序中使用 thymeleaf-extras-springsecurity3 或 thymeleaf-extras-springsecurity4 模块，我们首先需要以通常的方式为 Spring + Thymeleaf 的应用程序（*TemplateEngine* bean，*模板解析器*等）配置我们的应用程序，并将 SpringSecurity 方言添加到我们的模板引擎，以便我们可以使用 `sec:*` 属性和特殊表达式工具对象：

```xml
    <bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
      ...
      <property name="additionalDialects">
        <set>
          <!-- Note the package would change to 'springsecurity3' if you are using that version -->
          <bean class="org.thymeleaf.extras.springsecurity4.dialect.SpringSecurityDialect"/>
        </set>
      </property>
	  ...
    </bean>
```

A就这样！




	
使用表达式工具对象
------------------------------------

`#authentication` 对象可以很容易地使用，像这样：

```html
    <div th:text="${#authentication.name}">
        authentication 对象的 "name" 属性的值应显示在此处。
    </div>
```

`#authorization` 对象可以以类似的方式使用，它通常位于 `th:if` 或 `th:unless` 中：


```html
    <div th:if="${#authorization.expression('hasRole(''ROLE_ADMIN'')')}">
        只有认证用户具有角色 ROLE_ADMIN 时，才会显示此信息。
    </div>
```

`#authorization` 对象是 `org.thymeleaf.extras.springsecurity[3|4].auth.Authorization` 的一个实例，请参阅此类及其文档以了解所提供的所有方法。

	
	
使用属性
--------------------


使用 `sec:authentication` 属性等同于使用 `#authentication` 对象，但它使用的是它自己的属性：

```html
    <div sec:authentication="name">
        authentication 对象的 "name" 属性的值应显示在此处。
    </div>
```

`sec:authorize` 和 `sec:authorize-expr` 属性完全相同。当对一个 *Spring Security Expression* 求值时，它们等价一个计算 `#authorization.expression(...)` 表达式的 `th:if`。



```html
    <div sec:authorize="hasRole('ROLE_ADMIN')">
        只有认证用户具有角色 ROLE_ADMIN 时，才会显示此信息。
    </div>
```

这些在 `sec:authorize` 属性中的 *Spring Security Expressions* 实际上是 Spring EL 表达式，它们在 SpringSecurity 特定的根对象上求值，这个对象包含了诸如 `hasRole(...)`、`getPrincipal()` 等方法 

与正常的 Spring EL 表达式一样，Thymeleaf 允许您从它们访问一系列对象，包括上下文变量映射（`#vars` 对象）。 实际上，如果让你感觉更舒服，你可以用 `${...}` 包围你的访问表达式：


```html
    <div sec:authorize="${hasRole(#vars.expectedRole)}">
        仅当认证的用户具有由控制器计算的角色时，才会显示。
    </div>
```

记住，Spring Security 设置了一个特殊的面向安全（security-oriented）的对象作为表达式根，这就是为什么你不能在上面的表达式中直接访问 `expectedRole` 变量。 


另一种检查授权的方法是 `sec:authorize-url`，它允许您检查用户是否有权访问特定网址：


```html
    <div sec:authorize-url="/admin">
        如果认证用户可以调用 "/admin" URL 的话，才会显示此信息。
    </div>
```

要指定特定的 HTTP 方法，请执行：

```html
    <div sec:authorize-url="POST /admin">
        如果认证用户可以使用 POST HTTP 方法调用 "/admin" URL 的话，才会显示此信息。
    </div>
```

最后，还有一个属性用于使用 Spring Security 的*访问控制列表（Access Control Lists）*来检查授权，这需要指定一个领域对象及其所定义的*权限（permissions）*。


```html
    <div sec:authorize-acl="${obj} :: '1,3'">
        只有认证用户在上下文变量 "obj" 引用的领域对象上具有权限 "1" 和 "3"时才会显示。
    </div>
```

在此属性中，领域对象和权限规范都被认为是 thymeleaf 的 *标准表达式（Standard Expressions）*。
