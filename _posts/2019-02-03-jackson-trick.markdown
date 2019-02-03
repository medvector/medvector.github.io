---
layout: single
title:  "Adding Custom Properties Using Jackson MixIns"
date:   2019-02-03
categories: [programming, jackson]
---
[Jackson](https://github.com/FasterXML/jackson) is one of the standard solutions in Java world when 
you need to serialize/deserialize your classes. Not only it supports several different data formats, but
it's also very well designed, documented and easy to use.
I'm particularly fond of [mix-ins](https://github.com/FasterXML/jackson-docs/wiki/JacksonMixInAnnotations) feature.
Mix-ins provide a convenient way to customize class (de)serialization without modifying it.

Let me first show you a simple example how you can serialize your class to [YAML](https://yaml.org/) format
using Jackson.
Consider the following code:

{% highlight java %}
class MyClazz {
    public int a = 10;
    public int b = 15;
}  

class Main {
  public static void main(String[] args) throws JsonProcessingException {
      YAMLFactory yamlFactory = new YAMLFactory();
      ObjectMapper mapper = new ObjectMapper(yamlFactory);
      System.out.println(mapper.writeValueAsString(new MyClazz()));
      
      // Output:
      //---
      //a: 10
      //b: 15
  }
}
{% endhighlight %}

Let's suppose that we can't or don't want to modify <code>MyClazz</code>, but we want to skip <code>b</code> during serialization.
We can do it by adding a mix-in for this class:

{% highlight java %}
private abstract class MyClazzMixIn {
    int a;

    // Jackson annotation that tells serializer to skip field/method
    @JsonIgnore
    int b;
}

class Main {
  public static void main(String[] args) throws JsonProcessingException {
      YAMLFactory yamlFactory = new YAMLFactory();
      ObjectMapper mapper = new ObjectMapper(yamlFactory);
      mapper.addMixIn(MyClazz.class, MyClazzMixIn.class);
      System.out.println(mapper.writeValueAsString(new MyClazz()));
      
      
      // Output:
      //---
      //a: 10
  }
}
{% endhighlight %}

Notice that we add @JsonIgnore annotation in mix-in class which duplicates <code>MyClazz</code> definitions.
You actually don't need to duplicate all the definitions, we can use more Jackson magic instead, but I don't
want to go into the details.

Imagine now that you need to add custom information to your serialized object. In our example let's add <code>c</code>
property which equals <code>a + b</code>.

We can use [JsonAppend](https://github.com/FasterXML/jackson-databind/blob/master/src/main/java/com/fasterxml/jackson/databind/annotation/JsonAppend.java)
annotation for this purpose. It allows us to add "virtual" properties to our objects.

Let's rewrite our example:
{% highlight java %}
@JsonAppend(props = {
    @JsonAppend.Prop(value = MyWriter.class, name = "c", type = Integer.class)
  })
  private abstract static class MyClazzMixIn {
    int a;
    
    @JsonIgnore
    int b;
  }

  public static void main(String[] args) throws JsonProcessingException {
    YAMLFactory yamlFactory = new YAMLFactory();
    ObjectMapper mapper = new ObjectMapper(yamlFactory);
    mapper.addMixIn(MyClazz.class, MyClazzMixIn.class);
    System.out.println(mapper.writeValueAsString(new MyClazz()));

    // Output:
    //---
    //a: 10
    //c: 25
  }
{% endhighlight %}
We don't even have to modify <code>ObjectMapper</code> initialization! Our mix-in class still is the only
place containing all the information about <code>MyClazz</code> serialization. 

To complete this story I'll show an implementation of <code>MyWriter</code>. It's just a class that knows
how to evaluate the value of our "virtual" property given an instance of <code>MyClazz</code>:

{% highlight java %}
private static class MyWriter extends VirtualBeanPropertyWriter {
    public MyWriter() {
    }

    public MyWriter(BeanPropertyDefinition propDef,
                    Annotations contextAnnotations,
                    JavaType declaredType) {
      super(propDef, contextAnnotations, declaredType);
    }

    @Override
    protected Object value(Object bean, JsonGenerator gen, SerializerProvider prov) {
      MyClazz myClazz = (MyClazz)bean;
      return myClazz.a + myClazz.b;
    }

    @Override
    public VirtualBeanPropertyWriter withConfig(MapperConfig<?> config,
                                                AnnotatedClass declaringClass,
                                                BeanPropertyDefinition propDef,
                                                JavaType type) {
      return new MyWriter(propDef, declaringClass.getAnnotations(), type);
    }
  }
{% endhighlight %}
