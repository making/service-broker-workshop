## はじめてのService Broker


「Spring Boot」を使ってまずは一番簡単なService Brokerを作りましょう。

#### 参考資料

* Implement Service Broker with Spring Boot [[Slide](http://www.slideshare.net/makingx/implement-service-broker-with-spring-boot-cftokyo)]
* Introducing Spring Cloud Cloud Foundry Service Broker [[Blog](https://spring.io/blog/2016/06/07/introducing-spring-cloud-cloud-foundry-service-broker)]
* Extending the Platform with Spring Boot and Cloud Foundry [[Slide](http://www.slideshare.net/KennyBastani/extending-the-platform-with-spring-boot-and-cloud-foundry)] [[YouTube](https://www.youtube.com/watch?v=SRSVxtkW3eQ)]

###  「Spring Tool Suite」を利用した「Spring Boot」アプリケーションの雛形プロジェクト作成

Spring 謹製の「統合開発環境」(IDE)「Spring Tool Suite」( 以降、「STS」) を利用して「Spring Boot」アプリケーションを開発します。

#### 「STS」で「Spring Boot」プロジェクトを新規作成

「STS」には「Spring Starter Project」という「Spring Boot」プロジェクトの雛形を 簡単に作る仕組みが用意されています。今回はこれを使ってプロジェクトを新規作成します。

 「STS」のメニューから、「File」→「New」→「Spring Starter Project」を選択します。


 ![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d6451ddb-4f72-79f4-f899-cbfa1fd5cbf4.png)

**「File」→「New」→「Spring Starter Project」メニューを選択**

プロジェクト情報の入力画面が表示されます。「Name」に"first-service-broker"を入力し、その他入力項目はここではデフォルトのままにします。必要に応じて変更してください。「Next」をクリックしてください。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/1a0622c9-e3b6-1044-26a7-34d36abbda25.png)


**プロジェクト情報を入力**

利用するプロジェクトを選択する画面が表示されます。「Boot Version」に「1.3.x (xは最新版)」を設定れて、「Finish」をクリックしてください。  

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/90e28f0f-5475-06ea-1a8a-dfa086657cdd.png)

> **ノート**
>
> Spring Boot 1.4以上を使用する場合は、**「Java Buildpack」のバージョンが3.7以上**である必要があります。


**「Boot Version」を「1.3.x」にしれて、 「Finish」をクリック**

「Package Explorer」内に「first-service-broker プロジェクト」が現われます。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d1159f47-dde5-7de3-9f1b-030dfa62ca35.png)

**「Package Explorer」内に「first-service-broker  プロジェクト」が現われる**

### 「Spring Cloud Cloud Foundry Service Broker」の導入

Service Brokerを実装するための便利プロジェクトである「Spring Cloud Cloud Foundry Service Broker」を導入します。

`pom.xml`に次の依存ライブラリを追加してください。

``` xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-cloudfoundry-service-broker</artifactId>
			<version>1.0.0.RELEASE</version>
		</dependency>
```

このライブラリを使うことで、Serivce Brokerを作成するのに次の3つのクラスのみ実装すれば良くなります。

* `org.springframework.cloud.servicebroker.service.ServiceInstanceService`
* `org.springframework.cloud.servicebroker.service.ServiceInstanceBindingService`
* `org.springframework.cloud.servicebroker.model.Catalog`

#### `ServiceInstanceService`


`ServiceInstanceService`には`cf create-service`、`cf delete-service`などで呼ばれる次の4メソッドを実装すれば良いです。

``` java
@Component
public class MyServiceInstanceService implements ServiceInstanceService {

    @Override
    public CreateServiceInstanceResponse createServiceInstance(
            CreateServiceInstanceRequest createServiceInstanceRequest) {
        // cf create-service
        // スキーマ作成処理など
        return new CreateServiceInstanceResponse();
    }

    @Override
    public GetLastServiceOperationResponse getLastOperation(
            GetLastServiceOperationRequest getLastServiceOperationRequest) {
        // サービスインスタンスが非同期で作成される場合に、cf servicesまたはcf serviceで呼ばれる
        // 状態チェック
        return new GetLastServiceOperationResponse();
    }

    @Override
    public DeleteServiceInstanceResponse deleteServiceInstance(
            DeleteServiceInstanceRequest deleteServiceInstanceRequest) {
        // cf deleteで呼ばれる
        // スキーマ削除処理など
        return new DeleteServiceInstanceResponse();
    }

    @Override
    public UpdateServiceInstanceResponse updateServiceInstance(
            UpdateServiceInstanceRequest updateServiceInstanceRequest) {
        // cf update-serviceで呼ばれる
        // スキーマの更新処理など
        return new UpdateServiceInstanceResponse();
    }
}
```

Service Broker APIの仕様に関しては

* [Provisioning](https://docs.cloudfoundry.org/services/api.html#provisioning)
* [Asynchronous Operations](https://docs.cloudfoundry.org/services/api.html#asynchronous-operations) / [Polling Last Operation (async only)](https://docs.cloudfoundry.org/services/api.html#polling)
* [Updating a Service Instance](https://docs.cloudfoundry.org/services/api.html#updating_service_instance)
* [Deprovisioning](https://docs.cloudfoundry.org/services/api.html#deprovisioning)

を参照してください。

#### `ServiceInstanceBindingService`

`ServiceInstanceService`には`cf bind-service`、`cf servce-service`で呼ばれる次の2メソッドを実装すれば良いです。

``` java
@Component
public class MyServiceInstanceBindingService
		implements ServiceInstanceBindingService {

	@Override
	public CreateServiceInstanceBindingResponse createServiceInstanceBinding(
			CreateServiceInstanceBindingRequest createServiceInstanceBindingRequest) {
		// cf bind-serviceで呼ばれる
		// ユーザー作成処理など
		Map<String, Object> credentials = new LinkedHashMap<>();
		// credeintailsのMap(json)作成
		return new CreateServiceInstanceAppBindingResponse().withCredentials(credentials);
	}

	@Override
	public void deleteServiceInstanceBinding(
			DeleteServiceInstanceBindingRequest deleteServiceInstanceBindingRequest) {
		// cf unbind-serviceで呼ばれる
		// ユーザー削除処理など
	}
}

```


Service Broker APIの仕様に関しては

* [Binding](https://docs.cloudfoundry.org/services/api.html#binding)
* [Unbinding](https://docs.cloudfoundry.org/services/api.html#unbinding)

を参照してください。

#### `Catalog`

`/v2/catalog`で返却するJSONに相当する`Catalog`インスタンスを作成します。


``` java
    @Bean
    Catalog catalog() {
        return new Catalog(Collections.singletonList(new ServiceDefinition(
                "1234567890",
                "p-demo",
                "A demo service broker",
                true,
                false,
                Collections.singletonList(new Plan("free", "free", "free plan",
                        new HashMap<String, Object>() {{
                            put("costs", Collections.singletonList(new HashMap<String, Object>() {
                                {
                                    put("amount", Collections.singletonMap("usd", 0.0));
                                    put("unit", "MONTHLY");
                                }
                            }));
                            put("bullets", Arrays.asList("fake", "foo", "bar"));
                        }}, true)),
                Arrays.asList("tag A", "tag B", "tag C"),
                new HashMap<String, Object>() {{
                    put("displayName", "Fake");
                    put("longDescription", "Fake Service");
                    put("imageUrl", "https://www.cloudfoundry.org/wp-content/uploads/2015/10/CF_rabbit_Blacksmith_rgb_trans_back-269x300.png");
                    put("providerDisplayName", "pivotal");
                }},
                null,
                null
        )));
    }
```

上記の`Catalog`オブジェクトは次のJSONに相当します。

``` json
{
  "services": [
    {
      "id": "1234567890",
      "name": "p-demo",
      "description": "A demo service broker",
      "bindable": true,
      "plan_updateable": false,
      "plans": [
        {
          "id": "free",
          "name": "free",
          "description": "free plan",
          "metadata": {
            "costs": [
              {
                "amount": {
                  "USD": 0
                },
                "unit": "MONTHLY"
              }
            ],
            "bullets": [
              "fake",
              "foo",
              "bar"
            ]
          },
          "free": true
        }
      ],
      "tags": [
        "tag A",
        "tag B",
        "tag C"
      ],
      "metadata": {
        "longDescription": "This is a Fake Service",
        "documentationUrl": "http://docs.gopivotal.com/pivotalcf/",
        "providerDisplayName": "pivotal",
        "displayName": "Fake",
        "imageUrl": "https://www.cloudfoundry.org/wp-content/uploads/2015/10/CF_rabbit_Blacksmith_rgb_trans_back-269x300.png"
      },
      "requires": [],
      "dashboard_client": null
    }
  ]
}
```

Service Broker APIの仕様に関しては[Catalog Management](https://docs.cloudfoundry.org/services/api.html#catalog-mgmt)を参照してください。


### Service Brokerの実装

「first-service-broker プロジェクト」で何もしないFake Service Brokerを作成しましょう。


最終的には次のプロジェクト構造になります。

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/d6098c60-7f8b-ca0a-48b6-3039aa092fb0.png)


#### `ServiceInstanceService`の実装

`src/main/java/com/exmple/FakeServiceInstanceService.java`を作成して、次のコードを実装してください。

``` java
package com.example;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.servicebroker.model.*;
import org.springframework.cloud.servicebroker.service.ServiceInstanceService;
import org.springframework.stereotype.Component;

@Component
public class FakeServiceInstanceService implements ServiceInstanceService {
    private final Logger log = LoggerFactory.getLogger(FakeServiceInstanceService.class);

    @Override
    public CreateServiceInstanceResponse createServiceInstance(CreateServiceInstanceRequest req) {
        String serviceInstanceId = req.getServiceInstanceId();
        log.info("Create Service Instance({})", serviceInstanceId);
        return new CreateServiceInstanceResponse();
    }

    @Override
    public GetLastServiceOperationResponse getLastOperation(GetLastServiceOperationRequest req) {
        String serviceInstanceId = req.getServiceInstanceId();
        log.info("Get Last Service Operation({})", serviceInstanceId);
        return new GetLastServiceOperationResponse();
    }

    @Override
    public DeleteServiceInstanceResponse deleteServiceInstance(DeleteServiceInstanceRequest req) {
        String serviceInstanceId = req.getServiceInstanceId();
        log.info("Delete Service Instance({})", serviceInstanceId);
        return new DeleteServiceInstanceResponse();
    }

    @Override
    public UpdateServiceInstanceResponse updateServiceInstance(UpdateServiceInstanceRequest req) {
        String serviceInstanceId = req.getServiceInstanceId();
        log.info("Update Service Instance({})", serviceInstanceId);
        return new UpdateServiceInstanceResponse();
    }
}
```


#### `ServiceInstanceBindingService`の実装

`src/main/java/com/exmple/FakeServiceInstanceBindingService.java`を作成して、次のコードを実装してください。

``` java
package com.example;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.servicebroker.model.CreateServiceInstanceAppBindingResponse;
import org.springframework.cloud.servicebroker.model.CreateServiceInstanceBindingRequest;
import org.springframework.cloud.servicebroker.model.CreateServiceInstanceBindingResponse;
import org.springframework.cloud.servicebroker.model.DeleteServiceInstanceBindingRequest;
import org.springframework.cloud.servicebroker.service.ServiceInstanceBindingService;
import org.springframework.stereotype.Component;

import java.util.LinkedHashMap;
import java.util.Map;
import java.util.UUID;

@Component
public class FakeServiceInstanceBindingService implements ServiceInstanceBindingService {
	private final Logger log = LoggerFactory.getLogger(FakeServiceInstanceBindingService.class);

	@Override
	public CreateServiceInstanceBindingResponse createServiceInstanceBinding(CreateServiceInstanceBindingRequest req) {
		String serviceInstanceId = req.getServiceInstanceId();
		String bindingId = req.getBindingId();
		log.info("Create Service Binding({}) for Service({})", bindingId, serviceInstanceId);
		Map<String, Object> credentials = new LinkedHashMap<>();
		credentials.put("url", "http://example.com/" + UUID.randomUUID());
		credentials.put("username", UUID.randomUUID());
		credentials.put("password", UUID.randomUUID());
		return new CreateServiceInstanceAppBindingResponse().withCredentials(credentials);
	}

	@Override
	public void deleteServiceInstanceBinding(DeleteServiceInstanceBindingRequest req) {
		String serviceInstanceId = req.getServiceInstanceId();
		String bindingId = req.getBindingId();
		log.info("Delete Service Binding({}) for Service({})", bindingId, serviceInstanceId);
	}
}
```

#### `Catalog`の定義

`FirstServiceBrokerApplication.java`に`Catalog`を定義してください。

``` java
package com.example;

import java.util.Arrays;
import java.util.Collections;
import java.util.HashMap;
import java.util.UUID;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.servicebroker.model.Catalog;
import org.springframework.cloud.servicebroker.model.Plan;
import org.springframework.cloud.servicebroker.model.ServiceDefinition;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class FirstServiceBrokerApplication {

	public static void main(String[] args) {
		SpringApplication.run(FirstServiceBrokerApplication.class, args);
	}


	// 追加
	@Bean
	Catalog catalog() {
		return new Catalog(Collections
				.singletonList(new ServiceDefinition(UUID.randomUUID().toString(), "p-demo", "A demo service broker", true, false,
						Collections.singletonList(new Plan("free", "free", "free plan", new HashMap<String, Object>() {
							{
								put("costs", Collections.singletonList(new HashMap<String, Object>() {
									{
										put("amount", Collections.singletonMap("usd", 0.0));
										put("unit", "MONTHLY");
									}
								}));
								put("bullets", Arrays.asList("fake", "foo", "bar"));
							}
						}, true)), Arrays.asList("tag A", "tag B", "tag C"), new HashMap<String, Object>() {
							{
								put("displayName", "Fake");
								put("longDescription", "Fake Service");
								put("imageUrl",
										"https://www.cloudfoundry.org/wp-content/uploads/2015/10/CF_rabbit_Blacksmith_rgb_trans_back-269x300.png");
								put("providerDisplayName", "pivotal");
							}
						}, null, null)));
	}
}
```



#### 認証情報の設定

最後にService BrokerのBasic認証ユーザーを`src/main/resources/application.properties`に設定します。

``` properties
security.user.name=demo
security.user.password=demo
```



#### ローカルで起動

`FirstServiceBrokerApplication.java`を右クリック -> 「Run As」 -> 「Spring Boot App」

![image](https://qiita-image-store.s3.amazonaws.com/0/1852/187cef7f-0d41-97f8-f2b5-6782b5d6e2f8.png)



``` console

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.3.7.RELEASE)

2016-08-25 01:55:38.695  INFO 98894 --- [           main] c.example.FirstServiceBrokerApplication  : Starting FirstServiceBrokerApplication on jpxxmakitm1.corp.emc.com with PID 98894 (/Users/makit/Downloads/workspace/first-service-broker/target/classes started by makit in /Users/makit/Downloads/workspace/first-service-broker)
2016-08-25 01:55:38.699  INFO 98894 --- [           main] c.example.FirstServiceBrokerApplication  : No active profile set, falling back to default profiles: default
2016-08-25 01:55:38.785  INFO 98894 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@429bd883: startup date [Thu Aug 25 01:55:38 JST 2016]; root of context hierarchy
2016-08-25 01:55:40.583  INFO 98894 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2016-08-25 01:55:40.596  INFO 98894 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2016-08-25 01:55:40.597  INFO 98894 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.0.36
2016-08-25 01:55:40.696  INFO 98894 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2016-08-25 01:55:40.697  INFO 98894 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1917 ms
2016-08-25 01:55:40.981  INFO 98894 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'characterEncodingFilter' to: [/*]
2016-08-25 01:55:40.981  INFO 98894 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2016-08-25 01:55:40.982  INFO 98894 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2016-08-25 01:55:40.982  INFO 98894 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'requestContextFilter' to: [/*]
2016-08-25 01:55:40.983  INFO 98894 --- [ost-startStop-1] .e.DelegatingFilterProxyRegistrationBean : Mapping filter: 'springSecurityFilterChain' to: [/*]
2016-08-25 01:55:40.983  INFO 98894 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2016-08-25 01:55:41.271  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/css/**'], []
2016-08-25 01:55:41.271  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/js/**'], []
2016-08-25 01:55:41.272  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/images/**'], []
2016-08-25 01:55:41.272  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/**/favicon.ico'], []
2016-08-25 01:55:41.272  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/error'], []
2016-08-25 01:55:41.375  INFO 98894 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: OrRequestMatcher [requestMatchers=[Ant [pattern='/**']]], [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@4352e470, org.springframework.security.web.context.SecurityContextPersistenceFilter@272b6515, org.springframework.security.web.header.HeaderWriterFilter@28765805, org.springframework.security.web.authentication.logout.LogoutFilter@536916bb, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@44249d4f, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@6415c167, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@45be768e, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@7d994996, org.springframework.security.web.session.SessionManagementFilter@365628a6, org.springframework.security.web.access.ExceptionTranslationFilter@741669ea, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5211638f]
2016-08-25 01:55:41.591  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@429bd883: startup date [Thu Aug 25 01:55:38 JST 2016]; root of context hierarchy
2016-08-25 01:55:41.683  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/catalog],methods=[GET]}" onto public org.springframework.cloud.servicebroker.model.Catalog org.springframework.cloud.servicebroker.controller.CatalogController.getCatalog()
2016-08-25 01:55:41.686  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}/service_bindings/{bindingId}],methods=[PUT]}" onto public org.springframework.http.ResponseEntity<?> org.springframework.cloud.servicebroker.controller.ServiceInstanceBindingController.createServiceInstanceBinding(java.lang.String,java.lang.String,org.springframework.cloud.servicebroker.model.CreateServiceInstanceBindingRequest)
2016-08-25 01:55:41.686  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}/service_bindings/{bindingId}],methods=[DELETE]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.servicebroker.controller.ServiceInstanceBindingController.deleteServiceInstanceBinding(java.lang.String,java.lang.String,java.lang.String,java.lang.String)
2016-08-25 01:55:41.689  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}],methods=[PUT]}" onto public org.springframework.http.ResponseEntity<?> org.springframework.cloud.servicebroker.controller.ServiceInstanceController.createServiceInstance(java.lang.String,org.springframework.cloud.servicebroker.model.CreateServiceInstanceRequest,boolean)
2016-08-25 01:55:41.689  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}],methods=[DELETE]}" onto public org.springframework.http.ResponseEntity<?> org.springframework.cloud.servicebroker.controller.ServiceInstanceController.deleteServiceInstance(java.lang.String,java.lang.String,java.lang.String,boolean)
2016-08-25 01:55:41.690  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}],methods=[PATCH]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.servicebroker.controller.ServiceInstanceController.updateServiceInstance(java.lang.String,org.springframework.cloud.servicebroker.model.UpdateServiceInstanceRequest,boolean)
2016-08-25 01:55:41.690  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/v2/service_instances/{instanceId}/last_operation],methods=[GET]}" onto public org.springframework.http.ResponseEntity<?> org.springframework.cloud.servicebroker.controller.ServiceInstanceController.getServiceInstanceLastOperation(java.lang.String)
2016-08-25 01:55:41.695  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2016-08-25 01:55:41.696  INFO 98894 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2016-08-25 01:55:41.724  INFO 98894 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2016-08-25 01:55:41.725  INFO 98894 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2016-08-25 01:55:41.769  INFO 98894 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2016-08-25 01:55:41.929  INFO 98894 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2016-08-25 01:55:42.037  INFO 98894 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2016-08-25 01:55:42.044  INFO 98894 --- [           main] c.example.FirstServiceBrokerApplication  : Started FirstServiceBrokerApplication in 3.726 seconds (JVM running for 4.571)
```


`/v2/catalog`にアクセスしてください。


``` console
$ curl -u demo:demo localhost:8080/v2/catalog

{"services":[{"id":"5774f385-7269-45d0-ba55-31f949977488","name":"p-demo","description":"A demo service broker","bindable":true,"plan_updateable":false,"plans":[{"id":"free","name":"free","description":"free plan","metadata":{"costs":[{"amount":{"usd":0.0},"unit":"MONTHLY"}],"bullets":["fake","foo","bar"]},"free":true}],"tags":["tag A","tag B","tag C"],"metadata":{"longDescription":"Fake Service","providerDisplayName":"pivotal","displayName":"Fake","imageUrl":"https://www.cloudfoundry.org/wp-content/uploads/2015/10/CF_rabbit_Blacksmith_rgb_trans_back-269x300.png"},"requires":[],"dashboard_client":null}]}
```


#### Service Brokerのデプロイ

ターミナルで`first-service-broker`プロジェクトに行き、

``` console
$ ./mvnw clean package -DskipTests
```

で`target/demo-0.0.1-SNAPSHOT.jar`が作成されるので`cf push`します。


``` console
$ cf push first-service-broker-<yourname> -p target/demo-0.0.1-SNAPSHOT.jar -m 512m
```

次に、`cf create-service-broker`でService Brokerを自分のSpaceに登録します。


``` console
$ cf create-service-broker p-demo demo demo http://first-service-broker-<yourname>.<domain> --space-scoped
```

`cf marketplace`で`p-demo`が登録されていることを確認してください。

``` console
$ cf marketplace

service      plans        description
p-demo       free         A demo service broker
...
```

> **ノート**
> 
> 管理者権限がある場合は
>
> ``` console
> $ cf create-service-broker p-demo demo demo http://first-service-broker-<yourname>.<domain>
> $ cf enable-service-access p-demo [-p PLAN] [-o ORG]
>
> で、対象のOrganizationにService Brokerを公開することができます。
> ```
