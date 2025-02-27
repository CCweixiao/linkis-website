---
title: 数据源 Client SDK 的使用
sidebar_position: 7
---

> Linkis DataSource 提供了方便的JAVA和SCALA调用的接口，只需要引入linkis-datasource-client的模块就可以进行使用，

## 1. 引入依赖模块
```
 <dependency>
   <groupId>org.apache.linkis</groupId>
   <artifactId>linkis-datasource-client</artifactId>
   <version>${linkis.version}</version>
 </dependency>
如：
 <dependency>
   <groupId>org.apache.linkis</groupId>
   <artifactId>linkis-datasource-client</artifactId>
   <version>1.1.0</version>
 </dependency>
```

## 2. Scala测试代码
建立Scala的测试类LinkisDataSourceClientTest，具体接口含义可以见注释：
```java

import com.fasterxml.jackson.databind.ObjectMapper
import org.apache.linkis.common.utils.JsonUtils
import org.apache.linkis.datasource.client.impl.{LinkisDataSourceRemoteClient, LinkisMetaDataRemoteClient}
import org.apache.linkis.datasource.client.request.{CreateDataSourceAction, GetAllDataSourceTypesAction, MetadataGetDatabasesAction, UpdateDataSourceParameterAction}
import org.apache.linkis.datasourcemanager.common.domain.{DataSource, DataSourceType}
import org.apache.linkis.httpclient.dws.authentication.StaticAuthenticationStrategy
import org.apache.linkis.httpclient.dws.config.{DWSClientConfig, DWSClientConfigBuilder}

import java.io.StringWriter
import java.util
import java.util.concurrent.TimeUnit


object LinkisDataSourceClientTest {

  def main(args: Array[String]): Unit = {
    val clientConfig =DWSClientConfigBuilder.newBuilder
      .addServerUrl("http://127.0.0.1:9001") //set linkis-mg-gateway url: http://{ip}:{port}
      .connectionTimeout(30000) //connection timtout
      .discoveryEnabled(false) //disable discovery
      .discoveryFrequency(1, TimeUnit.MINUTES)  // discovery frequency
      .loadbalancerEnabled(true) // enable loadbalance
      .maxConnectionSize(5) // set max Connection
      .retryEnabled(false) // set retry
      .readTimeout(30000) //set read timeout
      .setAuthenticationStrategy(new StaticAuthenticationStrategy) //AuthenticationStrategy Linkis authen suppory static and Token
      .setAuthTokenKey("hadoop")  // set submit user
      .setAuthTokenValue("xxx") // set passwd or token
      .setDWSVersion("v1") //linkis rest version v1
      .build
     //init datasource remote client   
     val dataSourceClient = new LinkisDataSourceRemoteClient(clientConfig)
     //init metadata remote client
     val metaDataClient = new LinkisMetaDataRemoteClient(clientConfig)
    //get all datasource type
    testGetAllDataSourceTypes(dataSourceClient)
    //create kafka datasource
    testCreateDataSourceForKafka(dataSourceClient)
    //create es datasource
    testCreateDataSourceForEs(dataSourceClient)
    //update datasource parameter for kafka
    testUpdateDataSourceParameterForKafka(dataSourceClient)
    //update datasource parameter for es
    testUpdateDataSourceParameterForEs(dataSourceClient)
    //get hive metadata database list
    testMetadataGetDatabases(metaDataClient)
  }

def testGetAllDataSourceTypes(client:LinkisDataSourceRemoteClient): Unit ={
    val getAllDataSourceTypesResult = client.getAllDataSourceTypes(GetAllDataSourceTypesAction.builder().setUser("hadoop").build()).getAllDataSourceType
    System.out.println(getAllDataSourceTypesResult)
  }

def testCreateDataSourceForKafka(client:LinkisDataSourceRemoteClient): Unit ={
    val dataSource = new DataSource();
    val dataSourceType = new DataSourceType
    dataSourceType.setName("kafka")
    dataSourceType.setId("2")
    dataSourceType.setLayers(2)
    dataSourceType.setClassifier("消息队列")
    dataSourceType.setDescription("kafka")
    dataSource.setDataSourceType(dataSourceType)
    dataSource.setDataSourceName("kafka-test")
    dataSource.setCreateSystem("client")
    dataSource.setDataSourceTypeId(2l);

    val dsJsonWriter = new StringWriter

    val mapper = new ObjectMapper
    JsonUtils.jackson.writeValue(dsJsonWriter, dataSource)
    val map = mapper.readValue(dsJsonWriter.toString,new util.HashMap[String,Any]().getClass)
    val id = client.createDataSource(CreateDataSourceAction.builder().setUser("hadoop").addRequestPayloads(map).build()).getInsert_id
    System.out.println(id)

  }

def testCreateDataSourceForEs(client:LinkisDataSourceRemoteClient): Unit ={
    val dataSource = new DataSource();
    dataSource.setDataSourceName("es-test")
    dataSource.setCreateSystem("client")
    dataSource.setDataSourceTypeId(7l);
    val dsJsonWriter = new StringWriter
    val mapper = new ObjectMapper
    JsonUtils.jackson.writeValue(dsJsonWriter, dataSource)
    val map = mapper.readValue(dsJsonWriter.toString,new util.HashMap[String,Any]().getClass)
    val id = client.createDataSource(CreateDataSourceAction.builder().setUser("hadoop").addRequestPayloads(map).build()).getInsert_id
    System.out.println(id)

  }

def testUpdateDataSourceParameterForKafka(client:LinkisDataSourceRemoteClient): Unit ={
    val params = new util.HashMap[String,Any]()
    val connParams = new util.HashMap[String,Any]()
    connParams.put("brokers","127.0.0.1:9092")
    params.put("connectParams",connParams)
    params.put("comment","kafka data source")
  client.updateDataSourceParameter(UpdateDataSourceParameterAction.builder().setUser("hadoop").setDataSourceId("7").addRequestPayloads(params).build())
  }

def testUpdateDataSourceParameterForEs(client:LinkisDataSourceRemoteClient): Unit ={
    val params = new util.HashMap[String,Any]()
    val connParams = new util.HashMap[String,Any]()
    val elasticUrls = new util.ArrayList[String]()
    elasticUrls.add("http://127.0.0.1:9200")
    connParams.put("elasticUrls",elasticUrls)
    params.put("connectParams",connParams)
    params.put("comment","es data source")
    client.updateDataSourceParameter(UpdateDataSourceParameterAction.builder().setUser("hadoop").setDataSourceId("8").addRequestPayloads(params).build())
  }


 def testMetadataGetDatabases(client:LinkisMetaDataRemoteClient): Unit ={
  client.getDatabases(MetadataGetDatabasesAction.builder().setUser("hadoop").setDataSourceId(9l).setUser("hadoop").setSystem("client").build()).getDbs
  }
}
```

