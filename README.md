# java7to8-handbook

### 1.객체 생성과 파괴
- 객체를 만들어야 할 떄와 만들지 말아야 할때
- 올바른 개게 생성 방법과 불필요한 생성을 피하는 방법
- 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업

#### 1.1 생성자 대신 정적 팩터리 메서드를 고려하라

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
