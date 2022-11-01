# 목차

1. [문제 발생 - List 자료구조로 add 하는 코드에서 예외가 발생](#1-문제-발생---list-자료구조로-add-하는-코드에서-예외가-발생) <br/>
2. [불변 리스트를 반환하는 메서드들](#2-불변-리스트를-반환하는-메서드들) <br/>
3. [List.of() 와 Arrays.asList() 가 반환하는 불변 리스트들의 차이점](#3-listof-와-arraysaslist-가-반환하는-불변-리스트들의-차이점) <br/>
4. [Collections.unmodifiableList(List list) 메서드 - 가변 리스트를 불변 리스트로 변환하여 반환하는 메서드](#4-collectionsunmodifiablelistlist-list-메서드---가변-리스트를-불변-리스트로-변환하여-반환하는-메서드) <br/>
5. [문제 해결 - 불변 리스트를 가변 리스트로 변환](#5-문제-해결---불변-리스트를-가변-리스트로-변환) <br/>

<br/>

# [자바, Java] 우아한 테크 코스 5기 프리코스 1주차 - List로 add() 했을 때 UnsupportedOperationException 발생하는 이유

<br/>

> **문제 중에서 입력 값이 List 타입으로 들어오는 문제가 있다.**
>
> **그 들어온 값들을 이용해서 도메인에서 데이터를 가공하려고 하는데,**
>
> **뭔 일인지 UnsupportedOperationException 예외가 발생했다.**
>
> **그 원인과 해결법을 찾아보자.**

<br/>

## 1. 문제 발생 - List 자료구조로 add 하는 코드에서 예외가 발생

이 문제는 이번에 프리코스에서 구현한 로직으로 예시를 들면, 이번 프리코스의 유출이 좀 많이 될 수도 있기 때문에 그냥 직접 간단히 작성한 코드들로 예시를 들어보겠다.

```java
class Scratch {
    void run () throws IOException {
        final List<Integer> list = List.of(1, 2, 3, 4);
        list.add(5); // 에러
        System.out.println(list);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// Exception in thread "main" java.lang.UnsupportedOperationException
// 	at java.base/java.util.ImmutableCollections.uoe(ImmutableCollections.java:71)
//	at java.base/java.util.ImmutableCollections$AbstractImmutableCollection.add(ImmutableCollections.java:75)
//	at Scratch.run(scratch.java:23)
//	at Scratch.main(scratch.java:28)
```

<br/>

지금 이 예시에서 발생한 예외와 같은 예외이다. 이러한 예외가 발생한 이유는, List.of() 메서드는 변경할 수 없는 즉, 불변의 리스트를 반환하기 때문이다. 그래서 add() 메서드를 통해 해당 리스트에 요소를 추가하려고 할 때 예외가 발생하는 것이다.



사실 난 원래 List.of() 가 불변 컬렉션을 반환하는 것을 알고 있었다. 그리고 이 예시에선 왜 에러가 나는지 쉽게 파악할 수 있다. 하지만 프리코스 문제를 풀 당시엔, **'불변 리스트를 입력 값으로 전달하는 테스트 코드'** 와 **'도메인 객체'** 가 분리가 되어있다. 그래서 도메인으로 들어오는 리스트가 불변 리스트인걸 까먹고 한참 해맨 기억이 있다.



이참에 불변 리스트를 반환하는 메서드들과 그 메서드들의 로직들을 간단히 살펴 보면서, 왜 불변인지 알아보고자 한다.

<br/>

## 2. 불변 리스트를 반환하는 메서드들

내가 현재 알고있는 것만 말하자면 크게 **2가지** 가 있다. **List.of()** 메서드와 **Arrays.asList()** 메서드이다. 이 둘의 내부 로직을 살펴보고 둘의 차이점까지 알아보도록 하자.

### 1. List.of()

List 의 of() 메서드부터 한 계층씩 파고 들어가보자.

```java
public interface List<E> extends Collection<E> {
    ... 생략
        
    static <E> List<E> of(E e1, E e2, E e3, E e4) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4);
    }
    
    ... 생략
}
```

<br/>

여기서부터 한가지 파악할 수 있는 것은, **ImmutableCollections** 클래스의 **ListN** 이라는 내부 클래스를 가져오는 것을 보니, **ImmutableCollections 클래스가 이름의 뜻 그대로 불변 컬렉션들을 관리하는 클래스** 라는 것을 알 수 있다. **ImmutableCollections 클래스의 ListN 내부 클래스** 에 한번 더 들어가자.

ImmutableCollections 클래스 안에 굉장히 많은 로직들이 있지만, 지금 우리에게 필요한 로직들만 골라서 예시로 가져와 보겠다.

```java
class ImmutableCollections {
    static UnsupportedOperationException uoe() { return new UnsupportedOperationException(); }
    
    static abstract class AbstractImmutableList<E> extends AbstractImmutableCollection<E>
            implements List<E>, RandomAccess { // ListN 내부 클래스의 부모인 추상 클래스
        
        // 여기있는 List의 기능들은 전부 예외처리 된 모습이다.
        @Override public void    add(int index, E element) { throw uoe(); }
        @Override public boolean addAll(int index, Collection<? extends E> c) { throw uoe(); }
        @Override public E       remove(int index) { throw uoe(); }
        @Override public void    replaceAll(UnaryOperator<E> operator) { throw uoe(); }
        @Override public E       set(int index, E element) { throw uoe(); }
        @Override public void    sort(Comparator<? super E> c) { throw uoe(); }
    }
    
    static final class ListN<E> extends AbstractImmutableList<E> // 위의 AbstractImmutableList 내부 클래스를 상속받음
            implements Serializable { // List.of() 메서드가 최종적으로 가져온 ListN 내부 클래스이다.

        static final List<?> EMPTY_LIST = new ListN<>();

        @Stable
        private final E[] elements;

        @SafeVarargs
        ListN(E... input) {
            @SuppressWarnings("unchecked")
            E[] tmp = (E[])new Object[input.length];
            for (int i = 0; i < input.length; i++) {
                tmp[i] = Objects.requireNonNull(input[i]);
            }
            elements = tmp;
        }
    }
}
```

<br/>

위의 예시를 보면 알겠지만, **'ImmutableCollections 클래스의 ListN 내부 클래스'** 는 **'ImmutableCollections 클래스의 AbstractImmutableList 내부 클래스'** 를 상속받고 있다. 그리고 AbstractImmutableList 내부 클래스 안에서 오버라이딩 된 List 의 기능들을 보면 **add, remove, set 등등의 기능들이 전부 UnsupportedOperationException 예외처리가 된 모습** 을 볼 수 있다.

<br/>

### 2. Arrays.asList()

Arrays 의 asList() 메서드부터 한 계층씩 파고 들어가보자.

```java
public class Arrays {
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }

    private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable {
        
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            a = Objects.requireNonNull(array);
        }
        
        ... 생략
    }
}
```

<br/>

Arrays 클래스 안에 있는 ArrayList 내부 클래스를 생성해서 반환하는 모습을 볼 수 있다. 근데 이 내부 클래스에는 add() 메서드가 구현되어있지 않다. 바로 ArrayList 내부 클래스가 상속받고 있는 AbstractList 클래스로 들어가 보면 알 수 있다.

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    ... 생략
        
    public boolean add(E e) {
        add(size(), e); // 밑의 add() 메서드르 호출
        return true;
    }
    
    public void add(int index, E element) { // 예외 처리가 된 모습
        throw new UnsupportedOperationException();
    }
    
    ... 생략
}
```

<br/>

역시나 add() 메서드는 예외 처리가 되어있는 모습이다. 이로써 List.of() 와 Arrays.asList() 가 어떻게 불변 리스트를 반환하는지 살펴보았다. 하지만 이 두 메서드에는 결정적인 기능 차이가 존재한다.

<br/>

## 3. List.of() 와 Arrays.asList() 가 반환하는 불변 리스트들의 차이점

결론부터 말하자면, 불변이라는 개념에 더 가까운 쪽은 List.of() 메서드가 반환하는 리스트라고 볼 수 있다.

### 1. List.of()

위에서도 예시로 가져왔던, List.of() 가 반환하는 리스트 내부의 예외처리 로직을 다시 보자.

```java
class ImmutableCollections {
    static UnsupportedOperationException uoe() { return new UnsupportedOperationException(); }
    
    static abstract class AbstractImmutableList<E> extends AbstractImmutableCollection<E>
            implements List<E>, RandomAccess {
        
        @Override public void    add(int index, E element) { throw uoe(); }
        @Override public boolean addAll(int index, Collection<? extends E> c) { throw uoe(); }
        @Override public E       remove(int index) { throw uoe(); }
        @Override public void    replaceAll(UnaryOperator<E> operator) { throw uoe(); }
        @Override public E       set(int index, E element) { throw uoe(); }
        @Override public void    sort(Comparator<? super E> c) { throw uoe(); }
    }
}
```

<br/>

add, set, remove, sort 등등 내부의 요소가 조금이라도 바뀔만한 건덕지를 절대 주지 않는다. 그야말로 불변 그 자체라고 볼 수 있다.

<br/>

### 2. Arrays.asList()

위의 Arrays.asList() 가 반환하는 내부 클래스의 로직 예시에서 생략했었던 부분들 중 몇개의 기능 로직들을 가져와보자.

```java
public class Arrays {
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }

    private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable {
        
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            a = Objects.requireNonNull(array);
        }

        ... 생략

        @Override
        public E set(int index, E element) { // List.of() 에선 불가능했던 set() 가능
            E oldValue = a[index];
            a[index] = element;
            return oldValue;
        }
        
        ... 생략

        @Override
        public void replaceAll(UnaryOperator<E> operator) { // List.of() 에선 불가능했던 replaceAll() 가능
            Objects.requireNonNull(operator);
            E[] a = this.a;
            for (int i = 0; i < a.length; i++) {
                a[i] = operator.apply(a[i]);
            }
        }

        @Override
        public void sort(Comparator<? super E> c) { // List.of() 에선 불가능했던 sort() 가능
            Arrays.sort(a, c);
        }

        ... 생략
    }
}
```

<br/>

구현된 메서드들만 봐도 List.of() 와의 차이점이 꽤 보일 것이다. Arrays.asList() 는 add(), remove() 같은 리스트의 크기를 변경하는 기능만 예외 처리 했을 뿐, 그 외의 기능들은 모두 동작하는 모습을 볼 수 있다. List.of() 가 반환하는 리스트에선 전부 예외 처리 되었던 sort(), set() 등의 기능들을 쓸 수 있다.

<br/>

## 4. Collections.unmodifiableList(List list) 메서드 - 가변 리스트를 불변 리스트로 변환하여 반환하는 메서드

이참에 가변에서 불변으로 변환해주는 메서드에 대해서도 알아보자. **원래 가변이던 리스트를 불변 리스트로 변환하여 반환해주는 메서드** 이다. 사실 리스트뿐만 아니라 Set, Map 등등 리스트 외의 컬렉션들도 불변으로 반환해주는 메서드가 있지만, 지금은 리스트만 알아보도록 하자.



먼저 Collections.unmodifiableList(List list) 메서드의 내부 로직을 살펴보자

```java
public class Collections {
    
    // Collections.unmodifiableList(List list) 의 메서드
    public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return (list instanceof RandomAccess ?
                new UnmodifiableRandomAccessList<>(list) :
                new UnmodifiableList<>(list)); // Collections클래스의 내부 클래스인 UnmodifiableList을 생성해서 반환하는 모습이다.
    }

    
    // 위의 메서드에서 생성한 Collections 클래스의 내부 클래스인 UnmodifiableList
    static class UnmodifiableList<E> extends UnmodifiableCollection<E>
                                  implements List<E> {
        private static final long serialVersionUID = -283967356065247728L;

        final List<? extends E> list;

        UnmodifiableList(List<? extends E> list) { // 위에서 호출된 UnmodifiableList 내부 클래스의 생성자
            super(list);
            this.list = list;
        }
        
        public boolean equals(Object o) {return o == this || list.equals(o);}
        public int hashCode()           {return list.hashCode();}

        public E get(int index) {return list.get(index);}
        public E set(int index, E element) { // set() 예외처리
            throw new UnsupportedOperationException();
        }
        public void add(int index, E element) { // add() 예외처리
            throw new UnsupportedOperationException();
        }
        public E remove(int index) { // remove() 예외처리
            throw new UnsupportedOperationException();
        }
        public int indexOf(Object o)            {return list.indexOf(o);}
        public int lastIndexOf(Object o)        {return list.lastIndexOf(o);}
        public boolean addAll(int index, Collection<? extends E> c) { // addAll() 예외처리
            throw new UnsupportedOperationException();
        }

        @Override
        public void replaceAll(UnaryOperator<E> operator) { // replaceAll() 예외처리
            throw new UnsupportedOperationException();
        }
        @Override
        public void sort(Comparator<? super E> c) { // sort() 예외처리
            throw new UnsupportedOperationException();
        }
        
        ... 생략
    }
}
```

<br/>

역시나 Collections.unmodifiableList(List list) 가 최종적으로 반환하는 UnmodifiableList 내부 클래스의 로직을 보면, **리스트의 요소들을 변경할 수 있는 모든 기능들에 대해 예외처리를 해주는 모습** 을 볼 수 있다.

<br/>

## 5. 문제 해결 - 불변 리스트를 가변 리스트로 변환

원래는 객체지향 설계에 있어서 불변 리스트를 반환하는 경우가 더 많지만, 내가 마주친 문제의 상황에선 add() 기능을 써야하기에 가변 리스트로 변경을 해줘야 했다. 예시를 보자.

```java
class Scratch {
    void run () throws IOException {
        List<Integer> immutableList = List.of(1,2,3,4,5);
        final ArrayList<Integer> mutableList = new ArrayList<>(immutableList);
        
        mutableList.add(6);
        System.out.println(mutableList);
    }
    
    public static void main(String[] args) throws IOException {
        new Scratch().run();
    }
}

// 출력 결과
// [1, 2, 3, 4, 5, 6]
```

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    transient Object[] elementData;
    
    ... 생략
        
    public ArrayList(Collection<? extends E> c) { // 위에서 호출된 ArrayList 생성자
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            elementData = EMPTY_ELEMENTDATA;
        }
    }
    
    public boolean add(E e) { // add() 기능
        modCount++;
        add(e, elementData, size);
        return true;
    }
    
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
    
    ... 생략
}
```

<br/>

new ArrayList(Collection collection) 생성자를 통해 리스트를 가변으로 변환하여 반환받은 후, add() 기능이 정상적으로 작동 되는 모습을 볼 수 있다. ArrayList 클래스는 List 의 모든 기능이 동작 가능하다는 것을 모두가 잘 알고 있을 것이다.



이번 글에선 불변 리스트의 문제를 해결하는 과정에서 가변과 불변 리스트에 관해 최대한 파고 들어가 보았다. 역시 개발자의 실력 향상은 문제를 해결하려고 노력하는 것으로부터 시작된다는 것을 다시한번 느낀다.