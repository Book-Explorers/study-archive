## **AAA 패턴의 중요성과 올바른 동작 판단**

AAA 패턴을 사용하는 부분과 여러 개의 준비, 실행, 검증 구절을 피하는 방법, 텍스트 픽스처의 재사용법, 그리고 단위 테스트 명명법에 대한 내용이 도움이 된다.

- AAA 패턴을 사용하는데 있어서 특히 중요한 점은 특히 'when' 구절에서 어떤 동작을 테스트해야 하는지 정확하게 판단하는 것이다.

## **텍스트 픽스처의 재사용법**

단위 테스트는 **코드의 안정성과 유지보수성**을 높이는 중요한 부분 중 하나이다. 여러 개의 실행 구절이나 검증 구절이 존재할 경우, 통합 테스트로 치부되기 쉬우므로 이를 줄이는 노력이 필요하다는 점이 특히 중요하다.

또한, 테스트 픽스처를 재사용하는데 있어서 생성자보다는 비공개 팩토리 메서드를 사용하는 것이 좋다는 점도 중요한 인사이트다. 이는 테스트 코드의 가독성을 높이고 결합도를 낮추어 독립적인 테스트를 유지하는 데 도움이 됩니다.

- 테스트 픽스처를 재사용할 때 생성자 초기화와 비공개 팩토리 메서드의 선택 기준은 **결합도와 가독성**을 중점으로 두어야 합니다.
- **생성자 초기화**
  - **장점:** 간단하게 여러 테스트 케이스에서 동일한 초기화 코드를 실행.
  - **단점:** 모든 테스트 케이스가 같은 초기화 코드를 실행하므로, 테스트 케이스 간의 결합도가 높음. 한 테스트 케이스에서 초기화를 변경하면 다른 테스트에도 영향.
- **비공개 팩토리 메서드**
  - **장점:** 각 테스트 케이스마다 필요한 초기화를 수행할 수 있어 테스트 간의 독립성이 높음. 특정 테스트에서 초기화를 변경해도 다른 테스트에 영향을 주지 않음.
  - **단점:** 초기화를 수행하는 비공개 팩토리 메서드가 여러 개 생길 수 있고, 이로 인해 코드 중복이 발생.
- code
  order

  ```java
  // Order 클래스
  public class Order {
      private String orderId;
      private String customerName;
      private int totalAmount;

      public Order(String orderId, String customerName, int totalAmount) {
          this.orderId = orderId;
          this.customerName = customerName;
          this.totalAmount = totalAmount;
      }

      // Getter 생략
  }

  ```

  ```java
  public class OrderTestWithConstructor {

      @Test
      public void testOrderTotalAmountCalculation() {
          // Arrange
          // Order 클래스의 요구사항이 변경되서 변수가 추가된다면?
          // 이와 같이 생성자로 만든 인스턴스를 모두 수정하는 작업..
          Order order = new Order("1", "Jin", 100);

          // Act
          int totalAmount = order.getTotalAmount();

          // Assert
          assertEquals(100, totalAmount);
      }
  }

  ```

  ```java
  public class OrderTestWithFactoryMethod {

      @Test
      public void testOrderTotalAmountCalculation() {
          // Arrange
          Order order = createOrderWithTotalAmount(100);

          // Act
          int totalAmount = order.getTotalAmount();

          // Assert
          assertEquals(100, totalAmount);
      }

      private Order createOrderWithTotalAmount(double totalAmount) {
          return new Order("1", "jin", totalAmount);
      }
  }

  ```

## **매개변수화된 테스트의 적절한 사용**

매개변수화된 테스트에 대한 리팩터링도 좋은 접근 방법이지만, 매개변수의 개수가 많아질수록 코드를 이해하기 어려워질 수 있으므로 적절한 균형을 유지하는 것이 중요합니다.

- 매개변수의 개수가 지나치게 많으면 테스트 케이스의 목적을 파악하기 어려워질 수 있으므로, 동작이 복잡하지 않은 경우에만 사용하는 것이 좋습니다. 또한, 모든 테스트 케이스가 공통된 구조를 가지는 것이 아니라면 매개변수화된 테스트를 사용하지 않는 것이 더 나을 수 있습니다.
