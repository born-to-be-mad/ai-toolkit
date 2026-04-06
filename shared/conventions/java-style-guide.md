# Java Style Guide

Canonical conventions for Spring Boot services that follow hexagonal architecture.
Both Claude skills and Copilot instructions reference this file — update here, both agents pick it up.

> **Java version**: 25 — use records, sealed classes, pattern matching, switch expressions, text blocks freely.

---

## Hexagonal Architecture

### Package Structure

```
com.example.<service-name>
├── domain/
│   ├── model/          # Domain entities and value objects (Java records)
│   ├── port/
│   │   ├── in/         # Input ports — interfaces named *Facade
│   │   └── out/        # Output ports — interfaces named *Provider
│   ├── service/        # Domain services implementing *Facade interfaces, named *Service
│   └── exception/      # Domain-specific exceptions
└── infrastructure/
    ├── adapter/
    │   ├── in/
    │   │   └── rest/         # REST controllers implementing generated *Api interfaces, named *Controller
    │   └── out/
    │       ├── rest/         # Outbound HTTP clients
    │       └── persistence/  # Database / repository adapters
    ├── config/         # Spring @Configuration classes
    └── props/          # @ConfigurationProperties classes
```

### Layer Rules

| Layer | May import | Must NOT import |
|---|---|---|
| `domain/model`, `domain/port`, `domain/exception` | Nothing outside `domain/` | `infrastructure.*`, Spring, Lombok |
| `domain/service` | `domain/**` | `infrastructure.*` |
| `infrastructure/**` | `domain/**`, Spring, Lombok | Other `infrastructure/` adapters (cross-adapter calls go through domain ports) |

### Naming Conventions

| Role | Suffix | Example |
|---|---|---|
| Input port (use case interface) | `*Facade` | `BookingFacade` |
| Output port (external system interface) | `*Provider` | `BookingProvider` |
| Domain service (implements Facade) | `*Service` | `BookingService` |
| REST controller (implements generated Api) | `*Controller` | `BookingController` |
| MapStruct mapper (DTO ↔ domain) | `*Mapper` | `BookingDtoMapper` |

---

## Domain Models

Use **Java records** for all domain entities, DTOs, and value objects.
Lombok `@Builder(toBuilder = true)` enables immutable updates.

```java
// domain/model/Booking.java
@Builder(toBuilder = true)
public record Booking(
    String id,
    String customerId,
    String resourceId,
    Instant createdAt
) {}
```

- No Spring annotations in domain records or port interfaces
- No `null` checks in constructors — validate at the service boundary
- Sealed interfaces for discriminated unions (status, event type, etc.)

---

## Dependency Injection

Always use **constructor injection** via Lombok `@RequiredArgsConstructor`.
Never use `@Autowired` on fields.

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class BookingService implements BookingFacade {
    private final BookingProvider bookingProvider;
    private final Clock clock;          // always inject — never Instant.now()
}
```

---

## Time Handling

**Always inject `java.time.Clock`. Never call time constructors directly.**

| ❌ Forbidden | ✅ Correct |
|---|---|
| `Instant.now()` | `clock.instant()` |
| `LocalDateTime.now()` | `LocalDateTime.now(clock)` |
| `ZonedDateTime.now()` | `ZonedDateTime.now(clock)` |
| `new Date()` | `Date.from(clock.instant())` |
| `System.currentTimeMillis()` | `clock.millis()` |

Clock is injected as a Spring bean. In unit tests, spy it with `@Spy Clock clock = Clock.fixed(NOW, UTC)`.

---

## MapStruct Mappers

Every mapper **must** have `@BeanMapping(ignoreByDefault = true)` and an explicit `@Mapping` for every field.
Silent field drops are the #1 source of data-loss bugs.

```java
@Mapper(config = MapstructConfig.class)
public interface BookingDtoMapper {

    @BeanMapping(ignoreByDefault = true)
    @Mapping(target = "id",         source = "id")
    @Mapping(target = "customerId", source = "customerId")
    @Mapping(target = "resourceId", source = "resourceId")
    @Mapping(target = "createdAt",  source = "createdAt")
    BookingDto toDto(Booking domain);

    @BeanMapping(ignoreByDefault = true)
    @Mapping(target = "id",         source = "id")
    @Mapping(target = "customerId", source = "customerId")
    @Mapping(target = "resourceId", source = "resourceId")
    @Mapping(target = "createdAt",  source = "createdAt")
    Booking toDomain(BookingDto dto);
}
```

- All mappers annotated with `@Mapper(config = MapstructConfig.class)`
- Never use `@Mapping(target = "field", ignore = true)` as a shortcut for optional fields — list it explicitly

---

## REST Controllers

Controllers are **thin adapters**: translate HTTP ↔ domain only. No business logic.

```java
@RestController
@RequiredArgsConstructor
public class BookingController implements BookingApi {
    private final BookingFacade bookingFacade;
    private final BookingDtoMapper bookingMapper;

    @Override
    public ResponseEntity<BookingDto> getBooking(String bookingId) {
        Booking booking = bookingFacade.getBooking(bookingId);
        return ResponseEntity.ok(bookingMapper.toDto(booking));
    }
}
```

- Implements the generated `*Api` interface (OpenAPI-first)
- Returns `ResponseEntity<T>`
- Never puts `if/else` or calculations in a controller

---

## Logging

- Add `@Slf4j` **only when the class contains at least one `log.*` call** — never as a precaution
- Default level for general flow messages: **`log.debug`**
- Level guide:

| Level | When to use |
|---|---|
| `log.debug` | Normal flow: entering a method, key intermediate values, branch taken |
| `log.info` | Significant business events: resource created/updated/deleted, external call completed |
| `log.warn` | Unexpected but recoverable: retry triggered, fallback used, optional data missing |
| `log.error` | Unrecoverable failures: exception caught at boundary, external system unreachable |

- Log at the **boundary where the exception is handled**, not where it is thrown
- Include enough context to diagnose without reading the stack trace: IDs, counts, durations

```java
// ✅ correct
@Slf4j
public class BookingAdapter implements BookingProvider {
    public Optional<Booking> findById(String id) {
        log.debug("Looking up booking id={}", id);
        // ...
    }
}

// ❌ wrong — @Slf4j with no log calls
@Slf4j
public class BookingService implements BookingFacade { ... }
```

## Exception Handling

- Custom exceptions per domain concept: `BookingNotFoundException`, `ResourceUnavailableException`
- `@ControllerAdvice` maps domain exceptions to HTTP responses in `infrastructure/config/`
- Never catch and swallow — log at `error` level at the boundary where the exception is handled

---

## Unit Tests

**Stack**: JUnit 5 + Mockito + AssertJ + **Instancio**

```java
@ExtendWith(MockitoExtension.class)
class BookingServiceTest {

    @Mock  BookingProvider bookingProvider;
    @Spy   Clock clock = Clock.fixed(NOW, UTC);
    @InjectMocks BookingService bookingService;

    @Test
    @DisplayName("getBooking - happy path")
    void getBooking() {
        // given
        var booking = Instancio.create(Booking.class);
        when(bookingProvider.findById("id")).thenReturn(Optional.of(booking));
        // when
        var result = bookingService.getBooking("id");
        // then
        assertThat(result)
            .usingRecursiveComparison()
            .ignoringAllOverriddenEquals()
            .isEqualTo(booking);
    }

    @Test
    @DisplayName("getBooking - not found throws BookingNotFoundException")
    void getBooking_notFound() {
        // given
        when(bookingProvider.findById("x")).thenReturn(Optional.empty());
        // when
        var thrown = catchThrowable(() -> bookingService.getBooking("x"));
        // then
        assertThat(thrown).isInstanceOf(BookingNotFoundException.class)
            .hasMessageContaining("x");
    }
}
```

### Test Data — Instancio

| Use case | Pattern |
|---|---|
| All fields, auto-generated | `Instancio.create(Booking.class)` |
| Specific field value | `Instancio.of(Booking.class).set(field(Booking::id), "fixed-id").create()` |
| Collection | `Instancio.createList(Booking.class)` |

**Never** construct domain objects with `new Booking(...)` or manual builders in tests.

### Assertions

| Use case | Pattern |
|---|---|
| Deep object equality | `.usingRecursiveComparison().ignoringAllOverriddenEquals().isEqualTo(expected)` |
| Null completeness | `.hasNoNullFieldsOrProperties()` |
| Exception | `catchThrowable(() -> {...})` then `assertThat(thrown).isInstanceOf(...)` |
| Collection equality (ordered) | `.containsExactlyElementsOf(expected)` |
| Collection equality (unordered) | `.containsExactlyInAnyOrderElementsOf(expected)` |
| All elements complete | `.allSatisfy(it -> assertThat(it).hasNoNullFieldsOrProperties())` |

**Never** chain `assertThat(result.getField())` — always prefer deep comparison on the whole object.

---

## Integration Tests

**Stack**: JUnit 5 + Testcontainers + WireMock + AssertJ + Instancio

```java
class BookingIT extends AbstractIT {

    @Test
    @DisplayName("POST /api/v1/booking/bookings - creates booking with all valid fields")
    void createBooking_whenValidRequest_shouldReturnCreatedBooking() {
        // given
        var request = Instancio.create(CreateBookingRequest.class);
        // when
        var result = api.createBooking(request);
        // then
        assertThat(result)
            .hasNoNullFieldsOrProperties()
            .usingRecursiveComparison()
            .ignoringAllOverriddenEquals()
            .ignoringFields("version", "createdAt", "updatedAt")
            .isEqualTo(expectedFrom(request));
        assertThat(result.getCreatedAt()).isEqualTo(ClockFreezeConfig.NOW);
    }
}
```

- Every IT **extends `AbstractIT`** — Spring context spins up once per suite
- `ClockFreezeConfig.NOW` — fixed timestamp bean used for all time assertions
- Testcontainers: emulate databases and message brokers
- WireMock: stubs for all outbound HTTP calls
- Every REST endpoint must have at least one IT
- Name pattern: `*IT.java`

---

## OpenAPI (quick reference)

Full rules in `copilot/instructions/openapi-conventions.md`. Key points:

- Path pattern: `/api/v1/{domain}/{resource}[:{action}]`
- `:action` (colon prefix) for non-CRUD operations: `POST .../bookings:search`
- All YAML string values **single-quoted**: `'200':`, `$ref: '#/components/schemas/Booking'`
- Tags: `{Domain}{Resource}` PascalCase, one per operation
- `operationId`: camelCase verb+noun — `createBooking`, `searchBookings`

---

## General Style

- `@Slf4j` only when the class has actual `log.*` calls — see Logging section above
- `Optional` only as return type, never as field or parameter
- Max method ~20 lines; extract if longer
- Max class ~300 lines; split by responsibility if longer
- `UPPER_SNAKE_CASE` for constants
- Test class name = `{ClassUnderTest}Test` for unit, `{Feature}IT` for integration
- `@DisplayName` on **every** test method — describe the scenario in plain English
