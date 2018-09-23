It provides the calling of interface between different modules, trying to find the simplest and least problematic solution. All businesses should write their interfaces which to be exposed in the Common module (or Common package with independent businesses) .And then modules rely only on Common, not on each other. Bro will dynamically add the implementation of the interfaces, and get the corresponding instance(singleton) through getApi().

## Usage
### Expose interfaces
```
// Extends the interface of IBroApi written in the Common module first.
public interface IDataApi extends IBroApi{

    int getTestData1();

}

// Implement the interface and annotate the implementation class to expose. Implementation should be written in its own business module.
@BroApi("DataApi")
public class DataApiImpl implements IDataApi {

    @Override
    public int getTestData1() {
        return 66666;
    }

    // Provide an initial life cycle for the service when the Bro is initialized.
    @Override
    public void onInit() {

    }

// Providing a life cycle for the service which declares its dependencies on other services before calling onInit()
// and then performing onInit() sequentially after parsing out a dependency  tree.
// If there is a cycle dependency, it will throw exception at startup.
    @Override
    public List<Class<? extends IBroApi>> onEvaluate() {
        ArrayList<Class<? extends IBroApi>> depends = new ArrayList<>();
        depends.add(IPiApi.class);
        return depends;
    }
}
```

### Using a service
The Class passed into the interface is used to get the corresponding implementation for the interface. Generally, we only do a single mapping of the interface-implementation.(you can only get the first implementation if you have multiple implementations, but it's not usual in practical use)
```
Bro.getApi(IDataApi.class).getTestData1();
```

## Best implementation
### Extend the boundaries of navigation through BroApi services
We have discussed the reason why don't provide a Fragment, Service, etc. In fact, it's not just Fragment, Service. In most cases, modules also depend on smaller View level components. They may not be provided by a simple data interface, but they can be encapsulated and exposed through the interface of BroApi.
```
class DummyView extends View implements DummyAction {
      ...
}

interface IDummyApi extends IBroApi {
      DummyAction getDummyVIew();
}

class DummyApiImpl implements IDummyApi {
      public DummyAction getDummyVIew() {
             return new DummyView(mContext);
      }
}
```
As the example, we've encapsulated the operation of DummyView into a DummyAction and exposed it. The user only needs to convert it into a View when they need to do some operations related View. In most cases, we can continue to use DummyAction to do the DummyView operation.
Flexible use of return of interfaces can create some incredible effects in some special scenarios, and providing more possibilities for improving the efficiency and decoupling of modules.

-------------

It only provides global interceptions and monitoring callbacks. In essence, because intercept and monitoring belong to the structure of the basic logic, it is not appropriate to expose each business to register, which may cause problems such as false intercept. When using interceptors, you can customize the processing flow of Pipeline and divide interceptors into several, but the proposed way is get
them together.

## Usage
### Initialization
At Bro initialization, the implementation of the addition interceptor and monitor:
```
etc.
```
### Parameter description
- BroProperties,  divided into two parts: clazz is the actual class name of the annotation class; extraParam is a JSON string that contains any annotations(full class name) other than the beginning of @broxxx and its contents, as well as some other class-related parameters  collected by Bro during its compilation.

## Best practices
### Page permission validation (scene validation)
In order to attract users, many apps do not check logins on the first few pages until some important operations(such as logging into "favorites", "member purchase",etc.) that pop up the login box or upgrade to vip. At this time, we can define two annotations as @needlogin @needvip in the Common module, and then add the annotations in the corresponding page:
```
@NeedLogin
@NeedVip("1")
@BroActivity("TestActivity")
public class TestActivity extends Activity {...}
```
This code will create the extraParams of BroProperties as:
```
{
"com.example.package.NeedLogin": "",
"com.example.package.NeedVip": "1",
}

```
This allows you to check in the interceptor whether the Activity jump contains such descriptions (NeedLogin, NeedVip), and do some controlling and jumping to the corresponding logic for global page permissions (for example, jump to the login page after an interception).

### To solve the bug
Generally, we can preset an online configuration check in the interceptor.When we get a page bug and we can't find a way to fix it at once, we can turn off the access to a specific name page by pushing configuration, and display a customized 404 page. This method to deal with emergency is very effective, getting more time for actual bug fixes and reducing user loss.
At the same time, when a Native page has a corresponding Web implementation version(it is better to unify the Uri), we can also accomplish the graceful degradation of Native -> Web by push configuring when a Bug occurs in Native Activity. Thus, the bug will not affect the user's use.
