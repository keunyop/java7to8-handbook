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