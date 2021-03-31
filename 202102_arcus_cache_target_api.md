# Ability to Dynamically Update and Manage Cache Target API Lists of ARCUS Applications

<img src="/images/arcus_cache_target_medium_main.png"></img>

The ARCUS Common Module makes use of Spring AOP technology and provides the ability to easily apply ARCUS Cache in a declarative manner to the 
application's `cache target API` without code modification in Java applications. The methods of granting Annotation and specifying `cache target API` data in the
Property file are included in `cache target API` as an application of the ARCUS Cache. More details on the ARCUS Common Cache are described in the 
[ARCUS Common Cache Module Use with Basic Pattern Caching in Java Environment article](https://medium.com/jam2in/arcus-common-cache-module-with-basic-pattern-caching-in-java-environment-db88c4bf7585).


Application redeployment was needed, in the case of, updates for `cache target API` or cache application property are required from the cache application methods
(Annotation method, Property file method) of ARCUS Common Module. Due to the need to makes updates without application redeployment, in addition to the conventional
static Property file method, the dynamic Property management method was developed.

ARCUS metadata storage ZooKeeper keeps the `cache target API` data and we have made it possible to make updates on them if it's necessary. ARCUS Common Module with ZooKeeper
Watcher feature can lookup for `cache target API's` real-time updates and we have made it possible to make dynamic updates to the ARCUS Cache Application.

<img src="/images/arcus_cache_target_zookeeper.png"></img>

## Management of Cache Target API Data in ZooKeeper

ZooKeeper has a directory structure of key-value items called znode (ZooKeeper Node) that stores and manages desired data in it. Below-shown ZooKeeper's directory
structure describes how to store and manage `cache target API` data of the ARCUS Application. At the top of the structure, there come `/arcus_app/cache_target_list/`
directories followed by `/service-code/` znodes of ARCUS Cache Cluster. Znodes of `service code` approach each Application and `cache target API` list of each Application 
that uses the corresponding ARCUS Cache Cluster, in order to store and manage `cache target API data`.

<img src="/images/arcus_cache_target_zookeeper_structure.png"></img>

The `key` of the znode that stores the `cache target API` data can solely identify the `cache target API` by using a target name consisting of the
`'package-name.class-name.method-name'` and the value of the znode stores the JSON property as shown in the below example.

```
* JSON Property of Cache Target*/
{

/*
As a signature of Cache Target API's made by 'package-name.class-name.method-name'. 
*/

  "target":"com.service.BoardService.getBoard",
  
/*
Appoints the ARCUS Cache Key's prefix generated from Cache Target API.
*/ 

 "prefix":"BOARD",

/*
Appoints the ARCUS Cache Item's TTL (Time to Live) generated from Cache Target API.
*/ 

"expireTime":60,

/*
Specifies the key parameters (separated by commas) that will be used as ARCUS Cache Key generated from Cache Target API. 
*/

"keyParams":["bno"],

/*
Automatic creation of ARCUS Cache Keys generated from Cache Target API: if true automatically create a cache key using all parameters; if false keyParams must be specified.
*/

"keyAutoGeneration":false,

/*
Condition of whether to apply Arcus Cache on the Cache Target API
*/

"enable":true,
 
/*
Append the Arcus Cache Key creation time information after the Arcus Cache Key string. This is intended for to create different cache keys.
*/

  "keyDate":"KEY_DATE_NONE"
}
```


All `arcus_apps`, `cache_target_list`, `service_code` znodes that store `cache target API`s are all generated in persistent type and the znodes that store the `cache target APIs` data are created in `persistent and sequence types`. The reason why we use the *sequence type* is to easily determine creation, deletion, and other `cache target API` property updates. The `cache target API` property update operation - removes the existing znode and generates a new znode of the same key with updated properties, thereby causes the corresponding znode's sequence value to be increased. For instance, with the existence of `com.service.BoardService.getBoard-0000000000` key's znode that has a cache
property information (details) of `com.service.BoardService.getBoard`, and when making an update in that API's cache property (by removing the existing znode and 
generating a new znode at the same time) a new znode with `com.service.BoardService.getBoard-0000000001` key will be created. Therefore, the ARCUS Application lookups 
only the child nodes in `/arcus_apps/cache_target_list/service-code` znode and allows you to check creation, deletion, and update as the sequence value in its own` cache 
target API list`.

## Query Latest Cache Target API List from Cache Common Module (using ZooKeeper Watcher)

ARCUS Common Module reads `cache target API` list under the corresponding service code of ZooKeeper's `/arcus_apps/cache_target_list` directory and uses *cache item map*
to store and manage the corresponding `cache target API` list.

If the update occurs in the corresponding `cache target API` list, in order to receive real-time notifications, we register the ZooKeeper's Watcher to the corresponding 
`service code's` znode. When creation, deletion, or `cache target API's` property details updated, ZooKeeper's Watcher detects any occurred updates, lookups the latest 
`cache target API` list again, and compares the *previous* `cache target APIs'` *sequence value* with the *latest sequence value*, and identifies the updated `cache target API`. 
Updated JSON property of `cache target API` is read again from ZooKeeper and replaced in `cache item map`.

<img src="/images/arcus_cache_target_watcher_en.png"></img>

## Creation, Deletion, and Update of Cache Target API Properties

By directly using the ZooKeeper Command Line Interface tool command - `zkCli`, you can create, delete and update `cache target API` properties as shown below. 
In the example below, `-s` option is given to create a znode of sequence type. There can be an inconvenience to operate directly with `zkCli` command every time, 
hence it is much efficient to write and use a script of the corresponding operation.

```
// CREATION of Cache Target API
[ZK: localhost:2180(CONNECTED)]
create -s /arcus_apps/cache_target_list/service-code/com.service.BoardService.getBoard 
{
 "target":"com.service.BoardService.getBoard",
 "prefix":"BOARD",
 "expireTime":60,
 "keyParams":["bno"],
 "keyAutoGeneration":false,
 "enable":true,
 "keyDate":"KEY_DATE_NONE"
}

// DELETION of Cache Target API
[ZK: localhost:2180(CONNECTED)]
delete /arcus_apps/cache_target_list/service-code/com.service.BoardService.getBoard-0000000000

// UPDATE of Cache Target API
[ZK: localhost:2180(CONNECTED)]
create -s /arcus_apps/cache_target_list/service-code/com.service.BoardService.getBoard
{
 "target":"com.service.BoardService.getBoard",
 "prefix":"BOARD",
 "expireTime":60,
 "keyParams":["bno"],
 "keyAutoGeneration":false,
 "enable":true,
 "keyDate":"KEY_DATE_NONE"
}
[ZK: localhost:2180(CONNECTED)]
delete /arcus_apps/cache_target_list/service-code/com.service.BoardService.getBoard-0000000000
```

Performing creation, deletion, and update operations of `cache target API` directly using `zkCli` command can be uncomfortable and sometimes difficult for an operator.
On the webpage, to efficiently operate and manage ARCUS Cache Cluster (including ZooKeeper Ensemble) JaM2in currently developing ARCUS Operational Tool. 
In the *ARCUS Operational Tool*, we have developed a `cache target` feature to manage the Application's `cache target APIs`, as shown in the below example.

Below `cache target API` list of a specific service code is shown. In this scenario, our service code's name is a `test`. By clicking on a specific `cache target API`, 
you can check the detailed JSON property.

<img src="/images/202102_arcus_cache_target_arcus_admin_tool_1.png"></img>

The below image displays how to create a new `cache target API`. A user enters the property values that will be made as a JSON property and creates the corresponding 
`cache target API`. Relatively, delete and update operations of `cache target API` can be performed in the same manner.

<img src="/images/202102_arcus_cache_target_arcus_admin_tool_2.png"></img>

To recap briefly what has been discussed so far on ARCUS Application's `cache target API` management feature is as follows. When a user creates, deletes, or updates 
the `cache target APIs` by using the `cache target` feature of ARCUS Operational Tool, these changes are stored and managed by ZooKeeper Ensemble, where ZooKeeper Watcher
monitors these change events and passes them to the ARCUS Common Module. As explained earlier, ARCUS Common Module lookups for updated `cache target API`, manages 
them with `cache item map`. ARCUS Cache will be automatically applied only for `cache target APIs` that stored in the `cache item map`.

<img src="/images/arcus_cache_target_logic_en.png"></img>

## Application Method to the ARCUS Common Module

In order to apply ARCUS Common Module to the Java Application first, add the ARCUS Common Module dependency to the `pom.xml` file of the Java application.

```
<dependencies>
   ...
   <dependency>
      <groupId>com.jam2in.arcus</groupId>
      <artifactId>arcus-app-common</artifactId>
      <version>1.4.0</version>
   </dependency>
   ...
</dependencies>
```

Then, create the ARCUS property file `(arcus.properties)` as shown below.

```
# ZooKeeper Ensemble Address 
# Used to Connect  ARCUS and Renew Cache Target List 
arcus.address=1.2.3.4:2181,1.2.3.4:2182,1.2.3.4:2183

# Arcus service code
arcus.serviceCode=test

# Connection Pool size of Arcus Client 
arcus.poolSize=8

# Operation precessing time (milliseconds)
arcus.asyncOperationTimeout=700

# Global Prefix of Arcus Cache Key 
arcus.globalPrefix=RELEASE
```

Lastly, if you set up the Spring as shown below, the ARCUS Common Module application will be completed. The ARCUS Common Module makes it very easy for a user 
to apply ARCUS Cache and dynamically manage a `cache target API` list.

```
@Configuration
// Arcus Property Loading
@PropertySource("classpath:arcus.properties")
// @Aspect Use
@EnableAspectJAutoProxy(proxyTargetClass = true)
@Import(ArcusBeans.class)
public class ArcusConfiguration { 
 @Autowired
 private ArcusBeans arcusBeans;
 
 @Bean
 public static PropertySourcesPlaceholderConfigurer 
          propertySourcesPlaceholderConfigurer() {
  return new PropertySourcesPlaceholderConfigurer();
 }
 
 @Bean
 public ArcusStarter arcusStarter() {
  return new ArcusStarter(arcusBeans);
 } 
   
 @Bean
 public ArcusCacheAspect arcusAdvice() {
  return new ArcusCacheAspect(arcusBeans);
 }
}
```

## Conclusion

In summary, I've introduced and explained how to dynamically create, delete and update `cache target APIs` without a need for re-configuration of the ARCUS Application,
and how to store and manage `cache target APIs` using ZooKeeper. ARCUS Common Module through ZooKeeper Watcher detects real-time updates in the `cache target API lists` 
and always maintains the latest updated list of `cache target APIs`. In the future, to improve `cache target API` management features, the following works will be processed.

- HIT, MISS, RATIO indication of `cache target`,
- HISTORY feature after `cache target` was updated (e.g. records of by whom and when the cache target was updated),
- When creating `cache target`, ability to reflect it as JSON or a JSON file.













































