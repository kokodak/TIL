# 자동차 경주 코드리뷰 정리

## 1단계 PR 코드리뷰

### [1단계 PR 링크](https://github.com/woowacourse/java-racingcar/pull/522)

1. DTO를 사용해야 할까?

    - View에서 Model을 의존하지 않는다면, Model이 변경될 수 있는 여지를 없앨 수 있으니 좋다고는 생각합니다.

    - 반드시 DTO를 써야한다! 라는 건 없습니다. 상황에 맞게 판단해서 사용하면 됩니다. (도메인 자체가 가볍거나, 불변객체라 View에 넘겨줘도 큰 문제가 없는 경우에는 Model 자체를 넘겨줘도 괜찮습니다)

    - DTO를 사용할 경우, 오버엔지니어링이 될 수도 있는지 잘 판단하여 쓰는 것을 추천합니다.

2. 메서드 참조로 변경할 수 있다면 변경하자.

    ```java
    .orElseThrow(() -> new IllegalArgumentException());

    .orElseThrow(IllegalArgumentException::new);
    ```

3. 변수명, 메서드명을 신경쓰자.

    - 단수/복수를 신경쓰는 것만으로도 이해하기 쉬운 코드가 될 수 있습니다.

    - 변수명과 메서드명의 통일성을 지키면 가독성이 좋아집니다.

4. 스트링 반복 연산은 `StringBuilder` 혹은 `StringBuffer` 를 사용하자.

    - `String` 은 불변이기 때문에, 문자열 연산이 발생하면 그때마다 새로운 인스턴스를 생성하는 방식입니다. 때문에 반복적인 연산이 발생하는 환경에서는 비효율적입니다. 반면, `StringBuilder` 나 `StringBuffer` 는 가변이기 때문에 반복적인 연산이 발생한다해도 `String` 연산보다는 효율적입니다.

    - `StringBuilder` 는 멀티쓰레드 환경에서 Thread-safe 하지 않습니다.

    - `StringBuffer` 는 멀티쓰레드 환경에서 Thread-safe 합니다.

5. `@BeforeEach` 는 반복되는 코드를 먼저 실행시키기 때문에 테스트 메서드가 가벼워지지만, 의도가 명확하게 보이지 않는다면 오히려 반복 코드를 테스트 코드에 녹여주는 게 가독성이 더 좋아질 수도 있습니다.

6. 호출하는 메서드 순서대로 메서드 배치순서를 정렬해둔다면 가독성 측면에서 좋습니다.

7. 변수나 메서드명에 타입을 지정하는 방식은 좋지 않습니다. 변수나 메서드명이 타입에 의존하게 되면, 타입이 변경됐을 때 변수나 메서드명도 함께 변경되어야 하기 때문입니다.

    ```java
    List<Car> carList = getCarList(carNames); // 만약 List가 Set으로 변경된다면?
    Set<Car> carSet = getCarSet(carNames); // 코드를 변경시켜야 한다.
    ```

8. 개행 문자는 `"\n"` 이 아닌, `System.lineSeparator()` 를 사용하자.

    - 개행 문자는 OS 별로 조금씩 다릅니다. 예를 들면, 윈도우는 `"\r\n"`, 리눅스는 `"\n"` 입니다. 따라서 서로 다른 OS 에서 프로그램이 동작할 때 의도치 않은 결과가 발생할 수 있습니다.

    - `System.lineSeparator()` 는 프로그램이 실행되는 OS에 알맞는 개행 문자를 반환합니다. 따라서 OS가 달라도 동일한 결과를 기대할 수 있습니다.

9. 어떤 메서드에 대한 단위 테스트에서 다른 메서드에 의존하는 것은 좋지 않다.

    - A 라는 메서드에 대한 단위 테스트에서 B 메서드에 의존한다면, 다음과 같은 문제가 발생할 수 있습니다.

        - B 메서드를 수정할 시, 수정되지 않은 A 메서드의 단위 테스트가 터질 수 있다.

    - 따라서 단위 테스트를 작성할 때는, 가능한 독립적인 테스트를 작성해야 하며, 이를 지키기 위해서는 단위 테스트의 크기를 작게 유지하는 것이 도움이 됩니다.

10. 생성자나 정적 팩터리 메서드가 N개일 경우, 가장 기본적인 생성자(필드를 최대한 많이 받는 생성자)에 검증로직 (`validate`)을 넣어두고, 다른 생성자는 그 생성자를 통해 인스턴스를 생성하게 하는 것이 좋습니다. **(변경 포인트 최소화)**

    ```java
    // 안좋은 예시

    public Car(final String name, final int position) {
        validate(name);
        this.name = name;
        this.position = position;
    }

    public Car(final String name) {
        validate(name);
        this.name = name;
        this.position = 0;
    }
    ```
    ```java
    // 좋은 예시

    public Car(final String name, final int position) {
        validate(name);
        this.name = name;
        this.position = position;
    }

    public Car(final String name) {
        this(name, 0);
    }
    ```