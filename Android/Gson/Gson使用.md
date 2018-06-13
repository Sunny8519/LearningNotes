## Gson使用

### 1. Gson常用的转换

Gson常用的转换有如下几种：

- 使用Gson中的`add()`或者`addProperty()`方法添加属性的key-value值然后转为Json字符串
- Json字符串转数组
- 数组转Json字符串
- Json字符串转对象
- 对象转Json字符串
- Json字符串转List
- List转Json字符串
- Json字符串转Map
- Map转Json字符串
- 带有Class类型字段的Java对象转Json字符串
- Json字符串转带有Class类型字段的Java对象

#### 1.1 add()或者addProperty()

```java
public static void toJson() {
        JsonObject jsonObject = new JsonObject();
        //addProperty()方法能添加指定的字段和对应的值
        jsonObject.addProperty("name", "Jack");
        jsonObject.addProperty("age", 25);
        jsonObject.addProperty("address", "天津市南开区");
        jsonObject.addProperty("sex", Boolean.TRUE);

        JsonObject childObject = new JsonObject();
        childObject.addProperty("relative", "父子");
        childObject.addProperty("name", "Tom");
        childObject.addProperty("age", 2);

        //add()方法可以添加JsonObject对象
        jsonObject.add("son", childObject);

        Log.i(GsonLearning.class.getSimpleName(), jsonObject.toString());
    }
```

#### 1.2 Json字符串转数组

```java
public static void jsonToArray() {
        Gson gson = new Gson();
        String jsonArray = "[\"1\",\"2\",\"3\",\"4\",\"5\",\"6\"]";
//        String jsonArray = "[\"https://github.com/leavesC\",\"https://www.jianshu.com/u/9df45b87cfdf\",\"Java\",\"Kotlin\",\"Git\",\"GitHub\"]";

        //把整数Json字符串转为int型数组，其他类型的数组类似
        int[] values = gson.fromJson(jsonArray, int[].class);
        Log.i(GsonLearning.class.getSimpleName(), "Json转为字符串数组：");
        for (int value : values) {
            Log.i(GsonLearning.class.getSimpleName(), value + "");
        }
    }
```

#### 1.3 数组转Json字符串

```java
public static void arrayToJson() {
        String[] strings = {"Kotlin", "Java", "C", "C++", "Groovy"};

        Gson gson = new Gson();
        String jsonString = gson.toJson(strings);

        Log.i(GsonLearning.class.getSimpleName(), jsonString);
    }
```

#### 1.4 Json字符串转对象

```java
/*说明：
* 1.本地的Bean类中字段有五个，如果Json字符串中多了一个字段的key-value值，这不影响Gson去生成这个Bean类对象
* 2.如果Json字符串中的某个字段的key错了导致跟Bean类中的字段名对应不上，同样不影响Gson去生成这个Bean类对   * 象，但是Bean类中的该字段值会为null
* 3.如果Json字符串中比Bean类中少了一个字段，这也不影响Gson生成Bean类对象，只不过这个Bean类对应的字段会为   * null
*/
public static void jsonToObject() {
        String json = "{\"authCode\":\"pad_filterpress\",\"classObject\":\"com.cnmts.modules.filterPress.fragment.MonitoringFilterPressFragment\",\"favor\":false,\"moduleIcon\":2131231078,\"moduleId\":16,\"moduleName\":\"压滤\"}";

        Gson gson = new Gson();
        AuthBean authBean = gson.fromJson(json, AuthBean.class);
        Log.i(GsonLearning.class.getSimpleName(), authBean.toString());
    }

public static class AuthBean {
        private String authCode;
        private String moduleName;
        private long moduleId;
        private int moduleIcon;
        private boolean favor;
}
```

#### 1.5 对象转Json字符串

```java
/*说明：
* 1.Bean类中某个字段没有赋值，如果该字段是基本数据类型，转为Json字符串时就为该字段的初始化值，如果该字段是 *   类对象(比如String)，那么没有赋值，默认就是null，此时该字段是不会被转为Json字符串的，原来有5个字段的， *   转换后只有4个字段。
*/
public static void objectToJson() {
        AuthBean authBean = new AuthBean();
        authBean.setAuthCode("12344");
        authBean.setFavor(true);
        authBean.setModuleIcon(1243345);
        authBean.setModuleId(2);
        authBean.setModuleName("压滤");

        Gson gson = new Gson();
        String json = gson.toJson(authBean);

        Log.i(GsonLearning.class.getSimpleName(), json);
    }
```

#### 1.6 Json字符串转List

```java
public static void jsonToList() {
        String json = "[{\"authCode\":\"pad_filterpress\",\"favor\":false,\"moduleIcon\":2131231078,\"moduleId\":16,\"moduleName\":\"压滤\"}," +
                "{\"authCode\":\"pad-whirlcone\",\"favor\":true,\"moduleIcon\":2131231080,\"moduleId\":17,\"moduleName\":\"旋流器\"}]";

        Gson gson = new Gson();
        List<AuthBean> list = gson.fromJson(json, new TypeToken<List<AuthBean>>() {
        }.getType());

        for (AuthBean bean : list) {
            Log.i(GsonLearning.class.getSimpleName(), bean.toString());
        }
    }
```

#### 1.7 List转Json字符串

```java
public static void listToJson() {
        List<AuthBean> list = new ArrayList<>(2);
        for (int i = 0; i < 2; i++) {
            AuthBean authBean = new AuthBean();
            authBean.setAuthCode("12344" + i);
            authBean.setFavor(true);
            authBean.setModuleIcon(1243345);
            authBean.setModuleId(2);
            authBean.setModuleName(null);

            list.add(authBean);
        }

        Gson gson = new Gson();
        String json = gson.toJson(list);

        Log.i(GsonLearning.class.getSimpleName(), json);
    }
```

#### 1.8 Map转Json字符串

```java
public static void mapToJson() {
        Map<String, AuthBean> map = new HashMap<>();

        for (int i = 0; i < 2; i++) {
            AuthBean authBean = new AuthBean();
            authBean.setAuthCode("12344" + i);
            authBean.setFavor(true);
            authBean.setModuleIcon(1243345);
            authBean.setModuleId(2);
            authBean.setModuleName(null);

            map.put(i + "", authBean);
        }

        Gson gson = new GsonBuilder().enableComplexMapKeySerialization().create();
        String jsonMap = gson.toJson(map);
        Log.i(GsonLearning.class.getSimpleName(), jsonMap);
    }
```

#### 1.9 Json转Map

```java
public static void jsonToMap() {
        String json = "{\"0\":{\"authCode\":\"123440\",\"moduleId\":2,\"favor\":true,\"moduleIcon\":1243345},\"1\":{\"authCode\":\"123441\",\"moduleId\":2,\"favor\":true,\"moduleIcon\":1243345}}";

        Gson gson = new GsonBuilder().enableComplexMapKeySerialization().create();
        Map<String, AuthBean> map = gson.fromJson(json, new TypeToken<Map<String, AuthBean>>() {
        }.getType());

        for (String key : map.keySet()) {
            Log.i(GsonLearning.class.getSimpleName(), map.get(key).toString());
        }
    }
```

#### 1.10 带有Class类型字段的Java对象转Json

```java
public static class AuthBean {
        private String authCode;
        private String moduleName;
        private long moduleId;
        private int moduleIcon;
        private boolean favor;
        private Class classObject; //Class
}
//说明：如果有这样一个Java对象转为Json是怎样的呢？
public static void objectWithClassToJson() {
        AuthBean authBean = new AuthBean();
        authBean.setAuthCode("12344");
        authBean.setFavor(true);
        authBean.setModuleIcon(1243345);
        authBean.setModuleId(2);
        authBean.setModuleName("压滤");
        authBean.setClassObject(AuthBean.class);

        Gson gson = new Gson();
        String json = gson.toJson(authBean);
        Log.i(GsonLearning.class.getSimpleName(), json);
    }
```

上面这种转换会抛出异常，异常信息如下：
java.lang.UnsupportedOperationException: Attempted to serialize java.lang.Class: com.sunny.project.android_learning.GsonLearning$AuthBean. Forgot to register a type adapter?

从异常信息我们也能看出来，我们好像忘记注册一个adapter了，那我们再换一种方式：

```java
public static void objectWithClassToJson() {
        AuthBean authBean = new AuthBean();
        authBean.setAuthCode("12344");
        authBean.setFavor(true);
        authBean.setModuleIcon(1243345);
        authBean.setModuleId(2);
        authBean.setModuleName("压滤");
        authBean.setClassObject(AuthBean.class);

        Gson gson = new GsonBuilder().registerTypeAdapter(Class.class, new TypeAdapter<Class>() {
            @Override
            public void write(JsonWriter out, Class value) throws IOException {
                out.value(value.getName());
            }

            @Override
            public Class read(JsonReader in) throws IOException {
                Class clazz = null;
                try {
                    clazz = Class.forName(in.nextString());
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                }
                return clazz;
            }
        }).create();
        String json = gson.toJson(authBean);
        Log.i(GsonLearning.class.getSimpleName(), json);
    }
```

这样就可以顺利转换Class类对象了。

#### 1.11 Json字符串转带有Class类型字段的Java对象

```java
public static void jsonToObjectWithClass() {
        String json = "{\"authCode\":\"12344\",\"classObject\":\"com.sunny.project.android_learning.GsonLearning$AuthBean\",\"moduleName\":\"压滤\",\"favor\":true,\"moduleId\":2,\"moduleIcon\":1243345}";

        Gson gson = new GsonBuilder().registerTypeAdapter(Class.class, new TypeAdapter<Class>() {
            @Override
            public void write(JsonWriter out, Class value) throws IOException {
                out.value(value.getName());
            }

            @Override
            public Class read(JsonReader in) throws IOException {
                Class clazz = null;
                try {
                    clazz = Class.forName(in.nextString());
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                }
                return clazz;
            }
        }).create();
        AuthBean authBean = gson.fromJson(json, AuthBean.class);

        Log.i(GsonLearning.class.getSimpleName(), authBean.toString());
    }
```

同1.10，在包含有Class字段的Json字符串转为Bean类对象时，也需要注册Class对应的Adapter。



参考文章：[Android Gson使用详解](https://mp.weixin.qq.com/s/pjxRgKQW6EDdlK4rBbmtZQ)