---
title: "JVM kotlin static 의미"
categories:
  - android
tags:
  - kotlin
---
코틀린은 자바와 함께 안드로이드에서 주개발 언어로 채택하고 있다.
자바와 100% 호환되는 언어이지만 다른 부분이 있어, 별도의 추가 구현이 필요한 부분이 있다. 

기본적으로 코틀린에는 static 을 사용하지 않는다. 

나도 단순히 클래스내에서는 아래와 같이 object내에 추가하는 것으로 static이라고 생각했는데, 디컴파일해서 확인해보니 아니었다. 

INSTANCE 객체가 하나인 Singleton class와 동일하게 동작한다. 
즉 java에서는 ByteArrayConverter.INSTANCE.longToByte(xx)
이런 식으로 호출해야 한다. 

- 코틀린 object 작성
~~~kotlin
object ByteArrayConverter {
    fun longToBytes(x: Long): ByteArray {
        val buffer = ByteBuffer.allocate(java.lang.Long.BYTES)
        buffer.putLong(x)
        return buffer.array()
    }
~~~

- Decompile
~~~java
@NotNull
   public static final ByteArrayConverter INSTANCE;

   @NotNull
   public final byte[] longToBytes(long x) {
      ByteBuffer buffer = ByteBuffer.allocate(8);
      buffer.putLong(x);
      byte[] var10000 = buffer.array();
      Intrinsics.checkNotNullExpressionValue(var10000, "buffer.array()");
      return var10000;
   }
~~~

하지만 kotlin에서 static을 사용할 순 없지만, Singleton과  static은 다른 개념이니, 다른 방안을 마련해주고 있다. 

annotation을 통해서 가능한데, @JVMStatic annotation을 붙인다면 가능해진다. 

- 코틀린 object 작성
~~~kotlin
    @JvmStatic
    fun longToBytes(x: Long): ByteArray {
        val buffer = ByteBuffer.allocate(java.lang.Long.BYTES)
        buffer.putLong(x)
        return buffer.array()
    }
~~~

- Decompile
~~~java
   @JvmStatic
   @NotNull
   public static final byte[] longToBytes(long x) {
      ByteBuffer buffer = ByteBuffer.allocate(8);
      buffer.putLong(x);
      byte[] var10000 = buffer.array();
      Intrinsics.checkNotNullExpressionValue(var10000, "buffer.array()");
      return var10000;
   }
~~~