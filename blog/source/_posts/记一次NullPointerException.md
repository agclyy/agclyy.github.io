---
title: 记一次NullPointerException
date: 2019-05-18 16:19:04
categories: 问题排查
tags: 问题排查

---

记一次线上问题的坑，NullPointException

代码中age字段是Integer 可以传null，但set方法中用int基本类型接收的，虽然不知道这种诡异的代码是如何存在的，但实际线上就是存在了

下文的测试代码中 正常set null值，编译不会通过，但是添加三目运算的那种实现就可以了

```java
public class TestNullPointException {

    private  class Person {
        String name;
        Integer age;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public Integer getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
  
    @Test
    public void test(){
        Person person = new Person();
//        person.setAge(null);
        person.setAge("aa".equals("aa") ? null : 4);
        person.setName("aa");
    }

}
```

![https://ws1.sinaimg.cn/large/006tNc79gy1g35ktz0dhhj31cw09edhp.jpg](https://ws1.sinaimg.cn/large/006tNc79gy1g35ktz0dhhj31cw09edhp.jpg)