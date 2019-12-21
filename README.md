# java7to8-handbook

### 2.객체 생성과 파괴

- 객체를 만들어야 할 떄와 만들지 말아야 할때
- 올바른 개게 생성 방법과 불필요한 생성을 피하는 방법
- 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업

#### 2.1 생성자 대신 정적 팩터리 메서드를 고려하라

- 이름을 가질 수 있다.
- 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

★ 정적 팩터리 메서드 명명 방식

```java
// from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 매서드
Date d = Date.from(instant);

// of: 여러 매개변수를 받아 적합한 타입의 인스터스를 반환하는 집계 메서드
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

// valueOf: from과 of의 더 자세한 버전
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

// instance 혹은 getInstance: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
StackWalker luke = StackWalker.getInstance(options);

// create 혹은 newInstance: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
Object newArray = Array.newInstance(classObject, arrayLen);

// getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.
FileStore fs = Files.getFileStore(path);

// newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.
BufferedReader br = FIles.newBufferedReader(path);

// type: getType과 newType의 간결한 버전
List<Complaint> litany = Collections.list(legacyLitany);
```

예)

```java
// 기존 생성자 방식
public PublishQueryOrder(String tntInstId, String projectId) {
    this.setTntInstId(tntInstId);
    this.setProjectId(projectId);
}

// 생성자 방식 사용법
PublishQueryOrder object = new PublishQueryOrder(tntInstId, projectId);

//////////////////////////////////////

// 정적 팩터리 메서드 방식
public static PublishQueryOrder objForQueryByProjectId(String tntInstId, String projectId) {
    PublishQueryOrder obj = new PublishQueryOrder();
    obj.setTntInstId(tntInstId);
    obj.setProjectId(projectId);
    return obj;
}

// 정적 팩터리 메서드 방식 사용법
PublishQueryOrder object = PublishQueryOrder.objForQueryByProjectId(tntInstId, projectId);
```

#### 2.4 인스턴스화를 막으려거든 private 생성자를 사용하라

- private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

##### 예)

```java
public class ApplyDateUtil {

    // 기본생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private ApplyDateUtil() {
        throw new AssertionError();
    }

    ... // 나머지 코드는 생략
}
```

꼭 Assertion Error를 던질 필요는 없지만, 클래스 안에서 실수로라도 생성자를 호출하지 않도록 해준다.

#### 2.6 불필요한 객체 생성을 피하라

- 값비싼 객체를 재사용해 성능을 개선한다.

```java
// 기존 방식
public class ApplyDateUtil {
    public static Date getMinDay() {
        // 값비싼 객체 생성
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // Parse date format
        return sdf.parse(ProductValueRangesEnum.TIME.getMin());
    }

    public static Date getMaxDay() {
        // 값비싼 객체 생성
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // Parse date format
        return sdf.parse(ProdcoreConstants.LAST_DATE_TIME_STR);
    }
}

// 값비싼 객체 재사용 방식
public class ApplyDateUtil {
    
    // 값비싼 객체 생성은 캐싱하여 재사용 한다.
    private static final sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static Date getMinDay() {
        // Parse date format
        return sdf.parse(ProductValueRangesEnum.TIME.getMin());
    }

    public static Date getMaxDay() {
        // Parse date format
        return sdf.parse(ProdcoreConstants.LAST_DATE_TIME_STR);
    }
}

```

#### 2.9 try-finally 보다는 try-with-resources를 사용하라

```java
/**
 * 기존 try-finally 방식
 */
public class ProcessExecuteSql extends Thread {
    public void runSelect() {
        Connection con = null;
        Statement st = null;
        ResultSet rs = null;

        try {
            con = ConnectionManager.getConnection();
            st = con.createStatement();
            rs = st.executeQuery("");

        } catch (SQLException ex) {
            // 에러처리
        } finally {
            if (rs != null)
                try {
                    rs.close();
                } catch (SQLException ex) {
                }
            if (st != null)
                try {
                    st.close();
                } catch (SQLException ex) {
                }
            if (con != null)
                try {
                    con.close();
                } catch (SQLException ex) {
                }
        }
    }
}

/**
 * 권장 try-with-resources 방식
 */
public class ProcessExecuteSql extends Thread {
    public void runSelect() {

        try (Connection con = ConnectionManager.getConnection();
               Statement st = con.createStatement();
               ResultSet rs = st.executeQuery("")) {
        } catch (SQLException ex) {
            // 에러처리
        }
    }
}

```

### 4.클래스와 인터페이스

#### 4.19 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
```java
// 상속을 위한 메서드의 내부 동작 방식을 설명하는 주석
// Implementation Requirements
@implSpec
```

#### 4.21 인터페이스는 구현하는 쪽을 생각해 설계하라
```java
public interface ProdcoreHandleCallBack<T> {

    // 기존 방식
    public void checkParams();

    // 인터페이스에 추가된 디폴트 메서드 방식
    default void checkParams() {

    }
}
```

#### 4.22 인터페이스는 타입을 정의하는 용도로만 사용하라
```java
// 유틸리티성 클래스 (Constants 클래스도 포함) 는 인스턴스화할 수 없도록 한다.
public class DateUtil {
    // 기본생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private DateUtil() {
        throw new AssertionError();
    }
}
```

- 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트하여 클래스 이름을 생략할 수 있다.


### 5.제네릭

#### 5.27 비검사 경고를 제거하라

- @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자

```java
// @SuppressWarnings이 메소드 전체에 달려있다.
@SuppressWarnings({ "unchecked", "rawtypes" })
public PdLogHDO query(String tntInstId, String txDt, String txGuid, int seqNbr) {
    Map param = new HashMap();
    param.put("tntInstId", tntInstId);
    param.put("txDt", txDt);
    param.put("txGuid", txGuid);
    param.put("seqNbr", seqNbr);
}

// @SuppressWarnings 가능한 한 범위를 좁혀라.
public PdLogHDO query(String tntInstId, String txDt, String txGuid, int seqNbr) {
    @SuppressWarnings({ "unchecked", "rawtypes" })
    Map param = new HashMap();
    param.put("tntInstId", tntInstId);
    param.put("txDt", txDt);
    param.put("txGuid", txGuid);
    param.put("seqNbr", seqNbr);
}
```

#### 5.29 이왕이면 제네릭 타입으로 만들라

#### 5.33 타입 안전 이종 컨테이너를 고려하라

```java
// BEFORE
// ProductTableEnum의 getByCode() 함수를 많이 사용한다.
// 호출시마다 100개의 value를 매번 iterate 한다.
public static ProductTableEnum getByCode(String code) {
    for (ProductTableEnum value : ProductTableEnum.values()) {
        if (StringUtil.equals(code, value.getCode())) {
            return value;
        }
    }
    return null;
}

// AFTER
// ProductTableEnum의 각 value를 담고있는 Map을 만들어 tableNm을 key로 get하게끔 한다.
public class TableInfo {
    private static final Map<String, ProductTableEnum> tables = new HashMap<>();

    static {
        for (ProductTableEnum table : ProductTableEnum.values()) {
            tables.put(table.getCode(), table);
        }
    }

    public static ProductTableEnum getTable(String tableNm) {
        return tables.get(tableNm);
    }
}

// 호출방법
// BEFORE
ProductTableEnum table = ProductTableEnum.getByCode("테이블명");

// AFTER
ProductTableEnum table = TableInfo.getTable("테이블명");

```

### 7.람다와 스트림

#### 7.42 익명 클래스보다는 람다를 사용하라

```java
/**
 *  익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법
 */

// 조건코드 오름차순 정렬
Collections.sort(outList, new Comparator<ConditionTemplateBase>() {
    @Override
    public int compare(ConditionTemplateBase o1, ConditionTemplateBase o2) {
        return o1.getCode().compareTo(o2.getCode());
    }
});


/**
 *  람다식을 함수 객체로 사용 - 익명 클래스 대체
 */

// 조건코드 오름차순 정렬
Collections.sort(outList,
        (ConditionTemplateBase o1, ConditionTemplateBase o2) -> o1.getCode().compareTo(o2.getCode()));


/**
 *  Java 8에서 List 인터페이스에 추가된 sort 메서드를 이요하면 더욱 짧아짐
 */

// 조건코드 오름차순 정렬
outList.sort((ConditionTemplateBase o1, ConditionTemplateBase o2) -> o1.getCode().compareTo(o2.getCode()));

```

- 람다는 이름이 없고 문서화도 못 한다.
- 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
- 람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는게 좋다.


#### 7.43 람다보다는 메서드 참조를 사용하라
※ 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.


|메서드 참조 유형|예|같은 기능을 하는 람다|
|:-|:-|:-|
|정적|Integer::parseInt|str -> Integer.parseInt(str)|
|한정적(인스턴스)|Instant.now()::isAfter|Instant then = Instant.now(); t-> then.isAfter(t)|
|비한정적(인스턴스)|String::toLowerCase|str -> str.toLowerCase()|
|클래스 생성자|TreeMap<K, V>::new|() -> new TreeMap<K, V>()|
|배열 생성자|int[]::new|len -> new int[len]|

#### 7.46 스트림에서는 부작용 없는 함수를 사용하라
※ 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.

### 8. 매서드

#### 8.49 매개변수가 유효한지 검사하라

- 자바의 null 검사 기능 사용하기
```java
Objects.requireNonNull();
```

- public이 아닌 매서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다.
```java
assert a != null;
assert offset >= 0 && offset <= a.length;
assert length >= 0 && length <= a.length - offset;
```

#### 8.50 적시에 방어적 복사본을 만들라

- Date는 낡은 API이니 새로운 코드를 작성할 떄는 더 이상 사용하면 안된다. (Instant 나 LocalDateTime, ZOnedDateTime을 사용)

```java
// Period 인스턴스의 내부를 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다!
```

- 기존에 작성된 낡은 코드 대처 방법 => 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
}
```

※ 방어적 복사는 성능 저하가 따르므로 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다.

#### 8.51 매서드 시그니처를 신중히 설계하라

- 매서드 이름을 신중히 짖자. 같은 패키지에 속한 다른 이름들과 일관되게 짓는 게 최우선 목표
- 메서드를 너무 많이 만들지 말자.
- 매개변수 목록은 짭게 유지하자. 4개 이하가 좋다.