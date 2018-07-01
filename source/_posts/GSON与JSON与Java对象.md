---
title: GSON与JSON与Java对象
date: 2016-09-18 17:43:01
tags: [android,框架,Gson]
categories: [Android,Java]
---
## 为什么使用Gson
可能刚开始接触，会有个疑问，有了Json，为什么还会有一个Gson？Gson的存在，是不是多余的？
存在，即是有意义的。

Json可以将json转换为jsonObject，但当我们需要使用bean对象时，只能对jsonObject进行操作，这样对对象的操作是不是就不方便了呢，那么，使用Gson就很有必要了。这也是促使我去用它的一个原因。另一个原因就是Gson是google推荐的。

[gson-2.3.1.jar下载](http://download.csdn.net/download/u010637692/8348917)
[Gson项目实例](https://github.com/zouxiaobang/GsonProject)


## 使用
Gson gson = new Gson();

### 1、简单的转换
将对象转换为Json:
``` bash
Person person = new Person();
person.setName("John");
person.setAge(12);
person.setSex("man");

String json = mGson.toJson(person);
System.out.println(json);
```
结果：
I/System.out: {"sex":"man","name":"John","age":12}

将Json转换为对象:
json 为 {"sex":"man","name":"John","age":12}
``` bash
String json = "{\"sex\":\"man\",\"name\":\"John\",\"age\":12}";
Person person = mGson.fromJson(json, Person.class);

System.out.println(person.getName());
System.out.println(person.getAge() + "");
System.out.println(person.getSex());
System.out.println(person.doSome("run"));
```
结果：
I/System.out: John
I/System.out: 12
I/System.out: man
I/System.out: John run

通过将json转换为对象的结果可以看出Gson是将json所代表的对象转换过来，而不是JsonObject那样只能读取属性值，而不能操作方法。

### 2、List的泛型的转换
将List泛型的对象转换为json
``` bash
Person person1 = new Person("John", 12, "man");
Person person2 = new Person("May", 21, "woman");
Person person3 = new Person("David", 20, "man");
List<Person> persons = new ArrayList<>();
persons.add(person1);
persons.add(person2);
persons.add(person3);

String json = mGson.toJson(persons);
System.out.println(json);
```
该操作和上面的简单操作是一样的。
结果：
[{"sex":"man","name":"John","age":12},{"sex":"woman","name":"May","age":21},{"sex":"man","name":"David","age":20}]

将上面的结果作为json传入转换为List泛型的对象
``` bash
String json = "[{\"sex\":\"man\",\"name\":\"John\",\"age\":12}," +
                "{\"sex\":\"woman\",\"name\":\"May\",\"age\":21}," +
                "{\"sex\":\"man\",\"name\":\"David\",\"age\":20}]";

List<Person> persons = mGson.fromJson(json, new TypeToken<List<Person>>(){}.getType());
for (Person person: persons){
    System.out.println(person);
}
```
这里要将List泛型的类型作为第二个参数传进去，而List泛型获取参数的方法就是：new TypeToken<List<Person>>(){}.getType();
结果：
John : 12 : sex
May : 21 : sex
David : 20 : sex

### 3、Map的转换
将Map转换为Json：
``` bash
Map<String, Person> map = new HashMap<>();
map.put("a", new Person("John", 12, "man"));
map.put("b", new Person("May", 21, "woman"));
String json = mGson.toJson(map);

System.out.println(json);
```
结果：
I/System.out: {"b":{"sex":"woman","name":"May","age":21},"a":{"sex":"man","name":"John","age":12}}

以上面的json转换为Map：
``` bash
String json = "{\"b\":{\"sex\":\"woman\",\"name\":\"May\",\"age\":21}," +
                "\"a\":{\"sex\":\"man\",\"name\":\"John\",\"age\":12}}";
Map<String, Person> map = mGson.fromJson(json, new TypeToken<Map<String, Person>>(){}.getType());

for (String key: map.keySet()){
     System.out.println("key = " + key + ", value = " + ((Person)map.get(key)).doSome(" run"));
}
```
结果：
I/System.out: key = b, value = May  run
I/System.out: key = a, value = John  run

如果Map的第二个泛型是一个List泛型呢？那也一样，在转换成Map时强转map.get(key)为List泛型即可。








