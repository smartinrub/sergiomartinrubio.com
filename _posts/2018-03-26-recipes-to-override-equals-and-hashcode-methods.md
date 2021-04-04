---
title: Recipes to @Override “equals” and “hashCode” methods
image: https://lh3.googleusercontent.com/pw/ACtC-3fsJ5Uy_ZUq-vUJQBsgEaeXBlT_3oSGxSM5RInX02BB6JG1jkm5wrwG62gzLlEriElPnLCS1V_MHkJjO4yg5-DYA16_wdXOLGQVqTI-ZcQy3TA_LOxU6m0ouVrr1J4Iu644fc4_Next-BQm8WJpdYc9=w640-h423-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

## equals

- Use _==_ operator to compare object references. This is the most performance comparison.
- Use `instanceof` to check that both objects have same type.
- Cast the argument to the class type.
- Check desired fields in the class to return true when you consider that two objects are equals.
- To compare primitive fields whose type is not _double_ or _float_, use _==_ operator in the comparison.
- To compare objects, use equals method.
- To compare _float_ and _double_ use `Float.compare(float, float)` and `Double.compare(double, double)` respectively.
- To improve performance, first compare fields that are more likely to differ or less expensive to compare.
- Add test unless you autogenerate or use annotations.

```java
@Override 
public boolean equals(Object o) {

    if (o == this) return true;

    if (!(o instanceof MyObject)) return false;

    MyObject mo = (MyObject)o;

    return mo.mostImportantField == mostImportantField 
        && mo.sencondField == sencondField
        && mo.lessImportantField == lessImportantField;
}
```

## hashCode

- Declare an integer variable named `result` and initialize it with the first field.
- For primitive fields use e.g. `Integer.hashCode(field)`
- If the field is an object, call `hashCode` recursively.
- If the field is an array use `Arrays.hashCode`.
- Combine all hash codes into the `result` and return `result`.
- Multiply result every time by 31 (odd prime which can be replaced by a shift and a subtraction for better performance).

```java
@Override
public int hashCode() {
    int result = Integer.hashCode(firstField);
    result += 31 * result + sencondField.hashCode(); 
    result += 31 * result + Integer.hashCode(thirdField); 
    return result;
}
```


## Additional considerations

- Always override **HashCode** when overriding equals.
- Always use `Object` type as an argument in the equals method.
- Use `@Override` annotation.
- If there is no need to create a custom _equals_ or _hashCode_ method you only need to use [@Autovalue](https://github.com/google/auto/blob/master/value/userguide/index.md){:target="_blank"} or [@EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode){:target="_blank"} from **Google** and [**Lombok**](https://projectlombok.org/){:target="_blank"} libraries respectively.
- Alternatively, you can use autogeneration provided by your _IDE_.

Image by <a href="https://pixabay.com/photos/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=498199">Free-Photos</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=498199">Pixabay</a>
