# MapStruct Mapper Examples Guide

## Dependencies

```gradle
implementation 'org.mapstruct:mapstruct:1.5.5.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'
```

## Basic Mapper Interface

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDTO toDto(User user);
    User toEntity(UserDTO dto);
}
```

## Common MapStruct Annotations

### @Mapper
Marks an interface or abstract class as a mapper.

```java
@Mapper(
    componentModel = "spring",  // spring, cdi, jsr330, default
    unmappedTargetPolicy = ReportingPolicy.IGNORE,  // IGNORE, WARN, ERROR
    unmappedSourcePolicy = ReportingPolicy.WARN,
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface ProductMapper {
    ProductDTO toDto(Product product);
}
```

### @Mapping
Maps properties with different names or applies custom logic.

```java
@Mapper(componentModel = "spring")
public interface OrderMapper {
    
    @Mapping(source = "customer.name", target = "customerName")
    @Mapping(source = "orderDate", target = "purchaseDate")
    @Mapping(target = "totalAmount", expression = "java(order.getItems().stream().mapToDouble(Item::getPrice).sum())")
    @Mapping(target = "id", ignore = true)
    OrderDTO toDto(Order order);
}
```

### @BeanMapping
Configures the mapping method itself.

```java
@Mapper(componentModel = "spring")
public interface CustomerMapper {
    
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.SET_TO_NULL)
    CustomerDTO toDto(Customer customer);
    
    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateCustomerFromDto(CustomerDTO dto, @MappingTarget Customer customer);
}
```

## Advanced Examples

### 1. Date/Time Format Mapping

```java
@Mapper(componentModel = "spring")
public interface EventMapper {
    
    @Mapping(source = "eventDate", target = "date", dateFormat = "dd-MM-yyyy")
    @Mapping(source = "eventTime", target = "time", dateFormat = "HH:mm:ss")
    EventDTO toDto(Event event);
}
```

### 2. Number Format Mapping

```java
@Mapper(componentModel = "spring")
public interface PriceMapper {
    
    @Mapping(source = "price", target = "formattedPrice", numberFormat = "$#.00")
    @Mapping(source = "discount", target = "discountPercentage", numberFormat = "#.##%")
    PriceDTO toDto(PriceEntity price);
}
```

### 3. Collection Mapping

```java
@Mapper(componentModel = "spring", uses = {ProductMapper.class})
public interface ShoppingCartMapper {
    
    @Mapping(source = "items", target = "cartItems")
    ShoppingCartDTO toDto(ShoppingCart cart);
    
    // MapStruct automatically handles collection mapping
    List<ProductDTO> productsToProductDTOs(List<Product> products);
    Set<CategoryDTO> categoriesToCategoryDTOs(Set<Category> categories);
}
```

### 4. Nested Object Mapping with Cycle Prevention

```java
@Mapper(componentModel = "spring")
public interface DepartmentMapper {
    
    @Mapping(target = "employees", qualifiedByName = "toEmployeeDtoWithoutDepartment")
    DepartmentDTO toDto(Department department);
    
    @Named("toEmployeeDtoWithoutDepartment")
    @Mapping(target = "department", ignore = true)
    EmployeeDTO toEmployeeDtoWithoutDepartment(Employee employee);
}
```

### 5. Custom Mapping Methods

```java
@Mapper(componentModel = "spring")
public abstract class AddressMapper {
    
    @Mapping(target = "fullAddress", ignore = true)
    public abstract AddressDTO toDto(Address address);
    
    @AfterMapping
    protected void enrichDtoWithFullAddress(Address address, @MappingTarget AddressDTO dto) {
        dto.setFullAddress(address.getStreet() + ", " + 
                          address.getCity() + ", " + 
                          address.getCountry());
    }
}
```

### 6. Enum Mapping

```java
@Mapper(componentModel = "spring")
public interface StatusMapper {
    
    @ValueMapping(source = "PENDING", target = "IN_PROGRESS")
    @ValueMapping(source = "COMPLETED", target = "DONE")
    @ValueMapping(source = MappingConstants.ANY_REMAINING, target = "UNKNOWN")
    ExternalStatus toExternalStatus(InternalStatus status);
}
```

### 7. Conditional Mapping

```java
@Mapper(componentModel = "spring")
public interface UserProfileMapper {
    
    @Mapping(target = "email", expression = "java(user.isEmailPublic() ? user.getEmail() : null)")
    @Mapping(target = "phone", expression = "java(user.isPhonePublic() ? user.getPhone() : \"HIDDEN\")")
    UserProfileDTO toPublicProfile(User user);
}
```

### 8. Multiple Source Parameters

```java
@Mapper(componentModel = "spring")
public interface InvoiceMapper {
    
    @Mapping(source = "order.id", target = "orderId")
    @Mapping(source = "order.orderDate", target = "invoiceDate")
    @Mapping(source = "customer.name", target = "customerName")
    @Mapping(source = "customer.taxId", target = "customerTaxId")
    InvoiceDTO toInvoiceDto(Order order, Customer customer);
}
```

### 9. Update Existing Objects

```java
@Mapper(componentModel = "spring", 
        nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
public interface ProductUpdateMapper {
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    void updateProductFromDto(ProductUpdateDTO dto, @MappingTarget Product product);
}
```

### 10. Factory Method with @ObjectFactory

```java
@Component
public class EntityFactory {
    
    @ObjectFactory
    public Product createProduct(ProductDTO dto) {
        // Custom instantiation logic
        Product product = new Product();
        product.setSku(generateSku(dto));
        return product;
    }
    
    private String generateSku(ProductDTO dto) {
        return "PRD-" + System.currentTimeMillis();
    }
}

@Mapper(componentModel = "spring", uses = EntityFactory.class)
public interface ProductFactoryMapper {
    Product toEntity(ProductDTO dto);
}
```

### 11. Decorator Pattern

```java
@Mapper(componentModel = "spring")
@DecoratedWith(OrderMapperDecorator.class)
public interface OrderMapper {
    OrderDTO toDto(Order order);
}

public abstract class OrderMapperDecorator implements OrderMapper {
    
    @Autowired
    @Qualifier("delegate")
    private OrderMapper delegate;
    
    @Override
    public OrderDTO toDto(Order order) {
        OrderDTO dto = delegate.toDto(order);
        // Additional custom logic
        dto.setProcessedAt(LocalDateTime.now());
        return dto;
    }
}
```

### 12. Inheritance Configuration

```java
@MapperConfig(
    componentModel = "spring",
    unmappedTargetPolicy = ReportingPolicy.IGNORE,
    mappingInheritanceStrategy = MappingInheritanceStrategy.AUTO_INHERIT_FROM_CONFIG
)
public interface CentralConfig {
}

@Mapper(config = CentralConfig.class)
public interface InventoryMapper {
    InventoryDTO toDto(Inventory inventory);
}
```

### 13. Complex Nested Mapping Example

```java
@Mapper(componentModel = "spring", uses = {AddressMapper.class, ProductMapper.class})
public interface CompanyMapper {
    
    @Mapping(source = "headquartersAddress", target = "mainOffice")
    @Mapping(source = "employees", target = "staff")
    @Mapping(source = "products", target = "productCatalog")
    @Mapping(target = "revenue", expression = "java(company.getAnnualRevenue().toString() + \" USD\")")
    CompanyDTO toDto(Company company);
    
    @InheritInverseConfiguration
    @Mapping(target = "annualRevenue", expression = "java(new BigDecimal(dto.getRevenue().replace(\" USD\", \"\")))")
    Company toEntity(CompanyDTO dto);
}
```

### 14. Qualifying By Name for Ambiguous Mappings

```java
@Mapper(componentModel = "spring")
public interface PersonMapper {
    
    @Mapping(source = "birthDate", target = "age", qualifiedByName = "birthDateToAge")
    @Mapping(source = "address", target = "location", qualifiedByName = "addressToLocation")
    PersonDTO toDto(Person person);
    
    @Named("birthDateToAge")
    default Integer birthDateToAge(LocalDate birthDate) {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }
    
    @Named("addressToLocation")
    default String addressToLocation(Address address) {
        return address.getCity() + ", " + address.getCountry();
    }
}
```

### 15. Context Parameter Usage

```java
@Mapper(componentModel = "spring")
public interface LocalizedMapper {
    
    ProductDTO toDto(Product product, @Context Locale locale);
    
    @AfterMapping
    default void localize(Product product, @MappingTarget ProductDTO dto, @Context Locale locale) {
        // Apply localization based on context
        dto.setCurrency(Currency.getInstance(locale).getSymbol());
        dto.setDescription(getLocalizedDescription(product, locale));
    }
}
```

## Best Practices

1. **Use `unmappedTargetPolicy = ReportingPolicy.ERROR`** during development to catch unmapped properties early.

2. **Prefer `@Named` qualifiers** over `expression` for complex mappings for better readability and testability.

3. **Create separate mappers** for different concerns (e.g., `UserMapper`, `UserUpdateMapper`, `UserSummaryMapper`).

4. **Use `@MapperConfig`** for shared configuration across multiple mappers.

5. **Handle null values explicitly** with `nullValuePropertyMappingStrategy` and `nullValueCheckStrategy`.

6. **Test your mappers** - MapStruct generates code at compile time, so test the actual mapping logic.

7. **Use `uses` attribute** to compose mappers and avoid code duplication.

8. **Document complex mappings** with comments explaining the business logic.

## Complete Example: E-commerce Order System

```java
// Base mapper interface
public interface BaseMapper<E, D> {
    D toDto(E entity);
    E toEntity(D dto);
    void updateEntityFromDto(D dto, @MappingTarget E entity);
}

// Shared configuration
@MapperConfig(
    componentModel = "spring",
    unmappedTargetPolicy = ReportingPolicy.IGNORE,
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE
)
public interface MapperConfiguration {
}

// Product mapper
@Mapper(config = MapperConfiguration.class)
public interface ProductMapper extends BaseMapper<Product, ProductDTO> {
    
    @Override
    @Mapping(source = "category.name", target = "categoryName")
    @Mapping(source = "price", target = "formattedPrice", numberFormat = "$#,##0.00")
    ProductDTO toDto(Product entity);
    
    @Override
    @Mapping(target = "category", ignore = true)
    Product toEntity(ProductDTO dto);
}

// Order mapper with complex nested mappings
@Mapper(config = MapperConfiguration.class, uses = {ProductMapper.class, CustomerMapper.class})
public interface OrderMapper extends BaseMapper<Order, OrderDTO> {
    
    @Override
    @Mapping(source = "customer.fullName", target = "customerName")
    @Mapping(source = "orderItems", target = "items")
    @Mapping(target = "totalAmount", expression = "java(calculateTotal(entity))")
    @Mapping(source = "status", target = "statusDescription", qualifiedByName = "statusToDescription")
    OrderDTO toDto(Order entity);
    
    @Named("statusToDescription")
    default String statusToDescription(OrderStatus status) {
        return switch (status) {
            case PENDING -> "Order is being processed";
            case SHIPPED -> "Order has been shipped";
            case DELIVERED -> "Order has been delivered";
            case CANCELLED -> "Order was cancelled";
        };
    }
    
    default BigDecimal calculateTotal(Order order) {
        return order.getOrderItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

This guide covers the most common MapStruct patterns and annotations you'll encounter in real-world applications. Remember to always check the generated code in `target/generated-sources/annotations` to understand what MapStruct is creating for you.

## Spring Boot 3.4 Service Layer Integration

### 1. Basic Service Implementation

```java
@Service
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    public UserDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        return userMapper.toDto(user);
    }
    
    public List<UserDTO> findAll() {
        return userRepository.findAll().stream()
            .map(userMapper::toDto)
            .collect(Collectors.toList());
    }
    
    public UserDTO create(UserCreateDTO createDTO) {
        User user = userMapper.toEntity(createDTO);
        User savedUser = userRepository.save(user);
        return userMapper.toDto(savedUser);
    }
    
    public UserDTO update(Long id, UserUpdateDTO updateDTO) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("User not found"));
        userMapper.updateEntityFromDto(updateDTO, user);
        User updatedUser = userRepository.save(user);
        return userMapper.toDto(updatedUser);
    }
}
```

### 2. Transactional Service with Complex Mappings

```java
@Service
@Transactional
@RequiredArgsConstructor
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final CustomerRepository customerRepository;
    private final OrderMapper orderMapper;
    private final OrderItemMapper orderItemMapper;
    
    @Transactional(readOnly = true)
    public OrderDTO findOrderById(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
        return orderMapper.toDto(order);
    }
    
    @Transactional(readOnly = true)
    public Page<OrderSummaryDTO> findOrdersByCustomer(Long customerId, Pageable pageable) {
        Page<Order> orders = orderRepository.findByCustomerId(customerId, pageable);
        return orders.map(orderMapper::toSummaryDto);
    }
    
    public OrderDTO createOrder(OrderCreateDTO createDTO) {
        // Validate and fetch related entities
        Customer customer = customerRepository.findById(createDTO.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException(createDTO.getCustomerId()));
        
        // Create order entity
        Order order = new Order();
        order.setCustomer(customer);
        order.setOrderDate(LocalDateTime.now());
        order.setStatus(OrderStatus.PENDING);
        
        // Process order items
        List<OrderItem> orderItems = createDTO.getItems().stream()
            .map(itemDTO -> createOrderItem(itemDTO, order))
            .collect(Collectors.toList());
        
        order.setOrderItems(orderItems);
        order.setTotalAmount(calculateTotalAmount(orderItems));
        
        Order savedOrder = orderRepository.save(order);
        return orderMapper.toDto(savedOrder);
    }
    
    private OrderItem createOrderItem(OrderItemCreateDTO itemDTO, Order order) {
        Product product = productRepository.findById(itemDTO.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(itemDTO.getProductId()));
        
        OrderItem orderItem = orderItemMapper.toEntity(itemDTO);
        orderItem.setOrder(order);
        orderItem.setProduct(product);
        orderItem.setPrice(product.getPrice());
        
        return orderItem;
    }
    
    private BigDecimal calculateTotalAmount(List<OrderItem> items) {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

### 3. Service with Caching and Mapper Usage

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {
    
    private final ProductRepository productRepository;
    private final ProductMapper productMapper;
    private final CategoryService categoryService;
    
    @Cacheable(value = "products", key = "#id")
    public ProductDTO findById(Long id) {
        log.debug("Fetching product with id: {}", id);
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        return productMapper.toDto(product);
    }
    
    @Cacheable(value = "productsByCategory", key = "#categoryId")
    public List<ProductDTO> findByCategory(Long categoryId) {
        List<Product> products = productRepository.findByCategoryId(categoryId);
        return products.stream()
            .map(productMapper::toDto)
            .collect(Collectors.toList());
    }
    
    @CacheEvict(value = {"products", "productsByCategory"}, allEntries = true)
    public ProductDTO create(ProductCreateDTO createDTO) {
        Product product = productMapper.toEntity(createDTO);
        
        // Set category if provided
        if (createDTO.getCategoryId() != null) {
            Category category = categoryService.findEntityById(createDTO.getCategoryId());
            product.setCategory(category);
        }
        
        Product savedProduct = productRepository.save(product);
        return productMapper.toDto(savedProduct);
    }
    
    @CachePut(value = "products", key = "#id")
    @CacheEvict(value = "productsByCategory", allEntries = true)
    public ProductDTO update(Long id, ProductUpdateDTO updateDTO) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
        
        productMapper.updateEntityFromDto(updateDTO, product);
        Product updatedProduct = productRepository.save(product);
        return productMapper.toDto(updatedProduct);
    }
}
```

### 4. Service with Custom Mapper Logic

```java
@Service
@RequiredArgsConstructor
public class ReportService {
    
    private final OrderRepository orderRepository;
    private final ReportMapper reportMapper;
    
    public SalesReportDTO generateMonthlySalesReport(YearMonth yearMonth) {
        LocalDateTime startDate = yearMonth.atDay(1).atStartOfDay();
        LocalDateTime endDate = yearMonth.atEndOfMonth().atTime(23, 59, 59);
        
        List<Order> orders = orderRepository.findByOrderDateBetween(startDate, endDate);
        
        // Use mapper with context for complex report generation
        return reportMapper.toSalesReport(orders, yearMonth);
    }
    
    public CustomerReportDTO generateCustomerReport(Long customerId, DateRange dateRange) {
        Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new CustomerNotFoundException(customerId));
        
        List<Order> orders = orderRepository.findByCustomerAndDateRange(
            customer, dateRange.getStart(), dateRange.getEnd()
        );
        
        // Pass multiple parameters to mapper
        return reportMapper.toCustomerReport(customer, orders, dateRange);
    }
}

// Corresponding mapper
@Mapper(componentModel = "spring")
public interface ReportMapper {
    
    @Mapping(target = "period", source = "yearMonth")
    @Mapping(target = "totalOrders", expression = "java(orders.size())")
    @Mapping(target = "totalRevenue", expression = "java(calculateTotalRevenue(orders))")
    @Mapping(target = "orderSummaries", source = "orders")
    SalesReportDTO toSalesReport(List<Order> orders, YearMonth yearMonth);
    
    @Mapping(source = "customer.name", target = "customerName")
    @Mapping(source = "customer.email", target = "customerEmail")
    @Mapping(source = "orders", target = "orderHistory")
    @Mapping(source = "dateRange", target = "reportPeriod")
    @Mapping(target = "totalSpent", expression = "java(calculateTotalSpent(orders))")
    CustomerReportDTO toCustomerReport(Customer customer, List<Order> orders, DateRange dateRange);
    
    default BigDecimal calculateTotalRevenue(List<Order> orders) {
        return orders.stream()
            .map(Order::getTotalAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    default BigDecimal calculateTotalSpent(List<Order> orders) {
        return orders.stream()
            .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
            .map(Order::getTotalAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

### 5. Service with Validation and Error Handling

```java
@Service
@RequiredArgsConstructor
@Validated
public class CustomerService {
    
    private final CustomerRepository customerRepository;
    private final CustomerMapper customerMapper;
    private final Validator validator;
    
    public CustomerDTO create(@Valid CustomerCreateDTO createDTO) {
        // Additional business validation
        validateUniqueEmail(createDTO.getEmail());
        
        Customer customer = customerMapper.toEntity(createDTO);
        customer.setCreatedAt(LocalDateTime.now());
        customer.setActive(true);
        
        Customer savedCustomer = customerRepository.save(customer);
        return customerMapper.toDto(savedCustomer);
    }
    
    public CustomerDTO update(Long id, @Valid CustomerUpdateDTO updateDTO) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new CustomerNotFoundException(id));
        
        // Validate if email is being changed
        if (!customer.getEmail().equals(updateDTO.getEmail())) {
            validateUniqueEmail(updateDTO.getEmail());
        }
        
        customerMapper.updateEntityFromDto(updateDTO, customer);
        Customer updatedCustomer = customerRepository.save(customer);
        
        return customerMapper.toDto(updatedCustomer);
    }
    
    @Transactional(readOnly = true)
    public Page<CustomerDTO> search(CustomerSearchCriteria criteria, Pageable pageable) {
        Specification<Customer> spec = CustomerSpecifications.withCriteria(criteria);
        Page<Customer> customers = customerRepository.findAll(spec, pageable);
        
        return customers.map(customerMapper::toDto);
    }
    
    private void validateUniqueEmail(String email) {
        if (customerRepository.existsByEmail(email)) {
            throw new DuplicateEmailException("Email already exists: " + email);
        }
    }
}
```

### 6. Async Service with Mapper

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class NotificationService {
    
    private final NotificationRepository notificationRepository;
    private final NotificationMapper notificationMapper;
    private final EmailService emailService;
    
    @Async
    @Transactional
    public CompletableFuture<NotificationDTO> sendOrderConfirmation(Order order) {
        log.info("Sending order confirmation for order: {}", order.getId());
        
        Notification notification = new Notification();
        notification.setType(NotificationType.ORDER_CONFIRMATION);
        notification.setRecipient(order.getCustomer().getEmail());
        notification.setSubject("Order Confirmation #" + order.getId());
        notification.setContent(buildOrderConfirmationContent(order));
        notification.setSentAt(LocalDateTime.now());
        
        try {
            emailService.send(notification.getRecipient(), 
                            notification.getSubject(), 
                            notification.getContent());
            notification.setStatus(NotificationStatus.SENT);
        } catch (Exception e) {
            log.error("Failed to send notification", e);
            notification.setStatus(NotificationStatus.FAILED);
            notification.setErrorMessage(e.getMessage());
        }
        
        Notification saved = notificationRepository.save(notification);
        return CompletableFuture.completedFuture(notificationMapper.toDto(saved));
    }
    
    @Scheduled(fixedDelay = 300000) // 5 minutes
    @Transactional
    public void retryFailedNotifications() {
        List<Notification> failed = notificationRepository
            .findByStatusAndRetryCountLessThan(NotificationStatus.FAILED, 3);
        
        failed.forEach(this::retryNotification);
    }
    
    private void retryNotification(Notification notification) {
        notification.incrementRetryCount();
        try {
            emailService.send(notification.getRecipient(), 
                            notification.getSubject(), 
                            notification.getContent());
            notification.setStatus(NotificationStatus.SENT);
            notification.setSentAt(LocalDateTime.now());
        } catch (Exception e) {
            log.error("Retry failed for notification: {}", notification.getId(), e);
        }
        notificationRepository.save(notification);
    }
}
```

### 7. Service with Multiple Mapper Strategies

```java
@Service
@RequiredArgsConstructor
public class UserProfileService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final SecurityService securityService;
    
    /**
     * Returns different DTOs based on the viewer's relationship to the profile
     */
    public UserProfileDTO getProfile(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        
        Long currentUserId = securityService.getCurrentUserId();
        
        // Use different mapping strategies based on context
        if (userId.equals(currentUserId)) {
            // Full profile for own user
            return userMapper.toFullProfileDto(user);
        } else if (securityService.hasRole("ADMIN")) {
            // Admin view with sensitive data
            return userMapper.toAdminProfileDto(user);
        } else if (areFriends(currentUserId, userId)) {
            // Friend view with contact info
            return userMapper.toFriendProfileDto(user);
        } else {
            // Public view with limited info
            return userMapper.toPublicProfileDto(user);
        }
    }
    
    @Transactional
    public UserProfileDTO updateProfile(Long userId, ProfileUpdateDTO updateDTO) {
        if (!userId.equals(securityService.getCurrentUserId())) {
            throw new UnauthorizedException("Cannot update other user's profile");
        }
        
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        
        userMapper.updateProfileFromDto(updateDTO, user);
        user.setUpdatedAt(LocalDateTime.now());
        
        User updated = userRepository.save(user);
        return userMapper.toFullProfileDto(updated);
    }
    
    private boolean areFriends(Long userId1, Long userId2) {
        // Check friendship logic
        return userRepository.areFriends(userId1, userId2);
    }
}
```

### 8. Configuration Properties and Mapper Config

```java
@Configuration
@EnableCaching
@EnableAsync
@EnableScheduling
public class ServiceConfiguration {
    
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}

// Application properties for Spring Boot 3.4
```

```yaml
# application.yml
spring:
  application:
    name: mapstruct-demo
  
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
    open-in-view: false
  
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=500,expireAfterWrite=5m

logging:
  level:
    app.quantun.ecommerce.mapper: DEBUG
```

## Testing Service Layer with Mappers

```java
@SpringBootTest
@AutoConfigureMockMvc
class OrderServiceTest {
    
    @Autowired
    private OrderService orderService;
    
    @MockBean
    private OrderRepository orderRepository;
    
    @MockBean
    private OrderMapper orderMapper;
    
    @Test
    void testCreateOrder() {
        // Given
        OrderCreateDTO createDTO = OrderCreateDTO.builder()
            .customerId(1L)
            .items(List.of(
                OrderItemCreateDTO.builder()
                    .productId(1L)
                    .quantity(2)
                    .build()
            ))
            .build();
        
        Order order = new Order();
        order.setId(1L);
        
        OrderDTO expectedDTO = new OrderDTO();
        expectedDTO.setId(1L);
        
        when(orderRepository.save(any(Order.class))).thenReturn(order);
        when(orderMapper.toDto(order)).thenReturn(expectedDTO);
        
        // When
        OrderDTO result = orderService.createOrder(createDTO);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        verify(orderRepository).save(any(Order.class));
        verify(orderMapper).toDto(order);
    }
}
```

These examples demonstrate how MapStruct integrates seamlessly with Spring Boot 3.4 service layers, providing clean separation between entities and DTOs while maintaining type safety and performance.
