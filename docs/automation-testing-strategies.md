# 🧪 Testing Strategy for Clean Architecture

This document outlines the testing strategy for our clean architecture project, focusing on **unit tests** for business logic and **integration tests** for end-to-end API validation.

---

## **Unit Tests**

### **Purpose**
Unit tests validate the **business logic** within the `Application` layer, ensuring services, commands, and queries work as expected in isolation.

### **Project Structure**
- **Project Name**: `Application.UnitTests`
- **Framework**: xUnit
- **Mocking Library**: Moq
- **Key Practices**:
  - Mock all dependencies of the classes under test.
  - Follow the **AAA (Arrange, Act, Assert)** pattern for writing tests.
  - Use the naming convention:  
    ```
    MethodName_Condition_ExpectedResult
    ```

// Arrange, Act, Assert pattern
// Naming: MethodName_Condition_ExpectedResult

[Fact]
public void CalculateOrderTotal_ValidInput_ReturnsCorrectTotal()
{
    // Arrange
    var mockOrderRepository = new Mock<IOrderRepository>();
    mockOrderRepository.Setup(repo => repo.GetOrder(It.IsAny<Guid>()))
        .Returns(new Order { Items = new List<OrderItem> { new OrderItem { Price = 10, Quantity = 2 } } });

    var orderService = new OrderService(mockOrderRepository.Object);

    // Act
    var result = orderService.CalculateOrderTotal(Guid.NewGuid());

    // Assert
    Assert.Equal(20, result);
}

---

## **Integration Tests**

### **Purpose**
Integration tests validate the **entire system flow**, treating the API as a real client would. These tests ensure that all layers, including authorization and database interactions, work together seamlessly.

### **Project Structure**
- **Project Name**: `IntegrationTests`
- **Framework**: xUnit
- **HTTP Client**: Refit
- **External Dependencies**: WireMock

### **Key Practices**
1. **No Mocking of API Components**:  
   The API’s internal behavior (e.g., authorization, database) is not mocked.
2. **End-to-End Simulation**:  
   Tests mimic a real client application, using Refit to call API endpoints.
3. **WireMock for External Systems**:  
   A separate `WireMock` project is used to simulate external dependencies.

// BusinessPattern_UserAction_ExpectedResult

[Fact]
public async Task PlaceOrder_ValidRequest_ReturnsSuccessResponse()
{
    // Arrange
    var client = new HttpClient
    {
        BaseAddress = new Uri("http://localhost:5000") // Replace with actual API URL
    };
    var orderRequest = new
    {
        CustomerId = Guid.NewGuid(),
        Items = new[] { new { ProductId = Guid.NewGuid(), Quantity = 1 } }
    };

    // Act
    var response = await client.PostAsJsonAsync("/api/orders", orderRequest);

    // Assert
    response.EnsureSuccessStatusCode();
    var responseData = await response.Content.ReadAsStringAsync();
    Assert.Contains("OrderId", responseData);
}

---

## **WireMock Configuration**

### **Purpose**
WireMock is used to simulate external dependencies, ensuring integration tests focus on the API's behavior.

### **Features**
1. Define expected requests and responses for each external system.
2. Simulate delays or failures to test edge cases.

// Configure WireMock server to simulate external dependency
using WireMock.Server;
using WireMock.RequestBuilders;
using WireMock.ResponseBuilders;

public class WireMockSetup : IDisposable
{
    private readonly WireMockServer _server;

    public WireMockSetup()
    {
        _server = WireMockServer.Start(9091); // Replace with the desired port

        _server
            .Given(Request.Create()
                .WithPath("/external-api/resource")
                .UsingGet())
            .RespondWith(Response.Create()
                .WithStatusCode(200)
                .WithBody("{ \"message\": \"Success\" }"));
    }

    public void Dispose()
    {
        _server.Stop();
        _server.Dispose();
    }
}

---

## **Test Scope Summary**

### **Unit Tests**
- **Focus**: Isolated business logic
- **Mock Dependencies**: Yes
- **Frameworks**: xUnit, Moq
- **Naming Convention**:  
`MethodName_Condition_ExpectedResult`

### **Integration Tests**
- **Focus**: End-to-end API validation
- **Mock Dependencies**: No (except for external systems via WireMock)
- **Frameworks**: xUnit, Refit, WireMock
- **Naming Convention**:  
`BusinessPattern_UserAction_ExpectedResult`

---

## **General Guidelines**
1. **Testing Levels**: Focus primarily on the `Application` and `API` layers, as other layers are indirectly validated through integration tests.
2. **Tools**:
 - xUnit for test execution.
 - Moq for mocking dependencies in unit tests.
 - WireMock for simulating external systems in integration tests.
 - Refit for API communication during integration tests.
3. **Automation**: Ensure all tests are part of the CI/CD pipeline for consistent quality across deployments.

---

This approach ensures robust validation of both individual components and the entire system, maintaining reliability and scalability across all services.

---

## 🚀 Stay Connected
🔗 **Learn More:** [Your Website](https://cycolis-software.ro/home)  
💻 **Explore Our Work:** [GitHub](https://github.com/Cycolis-Software)  
💼 **Connect on LinkedIn:** [LinkedIn](https://www.linkedin.com/company/cycolis-software)  
🐦 **Follow for Updates:** [Twitter](https://x.com/CycolisSoftware) 
