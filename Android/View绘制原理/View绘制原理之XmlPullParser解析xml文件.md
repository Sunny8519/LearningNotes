## View绘制原理之XmlPullParser解析xml文件

#### 1.简易代码示例

PULL是Android自带的xml解析器，基于事件驱动。

示例代码：

```java
List<Person> persons = parseXml(getResources().getXml(R.xml.person));

private List<Person> parseXml(XmlResourceParser xmlResourceParser) {
        List<Person> persons = null;
        try {
            /*XmlPullParser parser = Xml.newPullParser(); //两种获取XmlPullParser对象的方式
            XmlPullParser parser = XmlPullParserFactory.newInstance().newPullParser();
            parser.setInput(in, "UTF-8");*/

            Person curPerson = null;
            int eventType = xmlResourceParser.getEventType();
            while (eventType != XmlPullParser.END_DOCUMENT) {
                String name = xmlResourceParser.getName();
                Log.i(TAG, eventType + ""); //1
                if (name != null)
                    Log.i(TAG, name); //2
                switch (eventType) {
                    case XmlPullParser.START_DOCUMENT:
                        persons = new ArrayList<>();
                        break;
                    case XmlPullParser.START_TAG:
                        if ("person".equalsIgnoreCase(name)) {
                            curPerson = new Person();
                            curPerson.setId(Integer.valueOf(xmlResourceParser.getAttributeValue(null, "id")));
                        } else if (curPerson != null) {
                            if ("name".equalsIgnoreCase(name)) {
                                curPerson.setName(xmlResourceParser.nextText());
                            } else if ("age".equalsIgnoreCase(name)) {
                                curPerson.setAge(Integer.valueOf(xmlResourceParser.nextText()));
                            }
                        }
                        break;
                    case XmlPullParser.END_TAG:
                        if ("person".equalsIgnoreCase(name) && curPerson != null && persons != null) {
                            persons.add(curPerson);
                            curPerson = null;
                        }
                        break;
                }
                eventType = xmlResourceParser.next();
            }
        } catch (IOException | XmlPullParserException e) {
            e.printStackTrace();
        }
        return persons;
    }
```

示例代码中使用`getResources.getXml(R.xml.person)`的方式获取XmlPullParser对象的，这种情况限于把要解析的xml文件放置在/res/xml/文件目录下，比如：

![xml.png](https://upload-images.jianshu.io/upload_images/5231076-95d1e54791e6af64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在注释处还提供了两种获取XmlPullParser对象的方式，分别为：

```java
XmlPullParser parser = Xml.newPullParser(); //方式一
XmlPullParser parser = XmlPullParserFactory.newInstance().newPullParser(); //方式二
```

通过这两种方式获取到XmlPullParser对象后，还要给XmlPullParser对象设置需要解析的文件的输入流才可，比如：

```java
XmlPullParser parser = Xml.newPullParser(); //方式一
parser.setInput(inputStream, "UTF-8");
```

其他操作都是相同的。

这是示例代码解析的xml文件内容：

```java
<persons>
    <person id="1">
        <name>张三</name>
        <age>30</age>
    </person>
    <person id="2">
        <name>李四</name>
        <age>25</age>
    </person>
</persons>
```

实体类Person如下：

```java
public class Person {
    private int id;
    private int age;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```



注意在示例代码中我用注释1和2标出了打印log的代码，我们通过log大致看一下解析的过程：

```java
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 0
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 0
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: persons  //a
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: person   //b
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: name     //d
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: age      //e
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 3
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: person   //b
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: person   //c
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: name
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 2
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: age
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 3
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: person  //c
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: 3
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: persons //a
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: Person{id=1, age=30, name='张三'}
04-12 18:05:56.734 16054-16054/com.sunny.learning I/XmlParserActivity: Person{id=2, age=25, name='李四'}
```

从日志上来看，a,b,c都是成对出现的，d和e却不是，persons和person标签都有对应的XmlPullParser.START_TAG和XmlPullParser.END_TAG事件，但是name和age标签却只有XmlPullParser.START_TAG事件。

下面给出对应事件的值：

```java
int START_DOCUMENT = 0;
int END_DOCUMENT = 1;
int START_TAG = 2;
int END_TAG = 3;
```

注意：解析的xml文件可以放置在如下地方

- /res/xml/文件夹下
- /res/raw/文件夹下
- /main/assets/文件夹下
- 放在外部存储器上

#### 2. 探索depth

```java
List<Person> persons = parseXml(getResources().getXml(R.xml.person));

private List<Person> parseXml(XmlResourceParser xmlResourceParser) {
        List<Person> persons = null;
        try {
            /*XmlPullParser parser = Xml.newPullParser(); //两种获取XmlPullParser对象的方式
            XmlPullParser parser = XmlPullParserFactory.newInstance().newPullParser();
            parser.setInput(in, "UTF-8");*/

            Person curPerson = null;
            int eventType = xmlResourceParser.getEventType();
            while (eventType != XmlPullParser.END_DOCUMENT) {
                String name = xmlResourceParser.getName();
                int depth = xmlResourceParser.getDepth(); //修改1
                Log.i(TAG, "EventType: " + eventType + " depth: " + depth); //修改2
                if (name != null)
                    Log.i(TAG, name);
                switch (eventType) {
                    case XmlPullParser.START_DOCUMENT:
                        persons = new ArrayList<>();
                        break;
                    case XmlPullParser.START_TAG:
                        if ("person".equalsIgnoreCase(name)) {
                            curPerson = new Person();
                            curPerson.setId(Integer.valueOf(xmlResourceParser.getAttributeValue(null, "id")));
                        } else if (curPerson != null) {
                            if ("name".equalsIgnoreCase(name)) {
                                curPerson.setName(xmlResourceParser.nextText());
                            } else if ("age".equalsIgnoreCase(name)) {
                                curPerson.setAge(Integer.valueOf(xmlResourceParser.nextText()));
                            }
                        }
                        break;
                    case XmlPullParser.END_TAG:
                        if ("person".equalsIgnoreCase(name) && curPerson != null && persons != null) {
                            persons.add(curPerson);
                            curPerson = null;
                        }
                        break;
                }
                eventType = xmlResourceParser.next();
            }
        } catch (IOException | XmlPullParserException e) {
            e.printStackTrace();
        }
        return persons;
    }
```

如上代码，对示例代码修改了1和2两处，打印结果如下：

```java
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 0 depth: 0
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 0 depth: 0
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 1
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: persons
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 2
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: person
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 3
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: name
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 3
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: age
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 3 depth: 2
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: person
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 2
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: person
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 3
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: name
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 2 depth: 3
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: age
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 3 depth: 2
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: person
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: EventType: 3 depth: 1
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: persons
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: Person{id=1, age=30, name='张三'}
04-12 18:16:44.594 17063-17063/? I/XmlParserActivity: Person{id=2, age=25, name='李四'}
```

对照着xml文件内容，可以看出解析根标签时，depth的值为1，伴随着解析的深入，depth值会增加，然后遇到结束标签时，depth值会相应的减小，直到根标签的结束标签，depth值又回到了1。其实depth值表示的就是xml文件中标签树的深度(maybe)。