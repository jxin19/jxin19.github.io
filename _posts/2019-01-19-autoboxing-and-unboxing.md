---
title: Autoboxing and Unboxing
description: Autoboxing and Unboxing
header: Autoboxing and Unboxing
---

Autoboxing과 Unboxing은 Java 5에서 소개되었다. 

## Autoboxing

기본 데이터 유형(Primitive type)을 Wrapper 클래스로 변환하는 것을 말한다. 예를 들어 int를 Integer로 변환하거나 long을 Long으로 변환하는 것이다.

## Unboxing

Wrapper 클래스의 유형을 다시 기본 데이터 유형으로 되돌려 놓는 것을 Unboxing이라 한다.

```java
import java.util.ArrayList;
import java.util.List;

public class AutoboxingUnboxing {

	public static void main(String[] args) {
		int i = 5;
		long j = 105L;

		// passed the int, will get converted to Integer object at Runtime using
		// autoboxing in java
		doSomething(i);

		List<Long> list = new ArrayList<>();

		// java autoboxing to add primitive type in collection classes
		list.add(j);
	}

	private static void doSomething(Integer in) {
		// unboxing in java, at runtime Integer.intValue() is called implicitly to
		// return int
		int j = in;

		// java unboxing, Integer is passed where int is expected
		doPrimitive(in);
	}

	private static void doPrimitive(int i) {

	}
}
```

<br/>

| Primitive type | Wrapper class |
| --- | --- |
| boolean | Boolean |
| byte | Byte |
| char | Character |
| float | Float |
| int | Integer |
| long | Long |
| short | Short |
| double | Double |

> 참고<br/>
> https://www.journaldev.com/1005/autoboxing-java
> https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
