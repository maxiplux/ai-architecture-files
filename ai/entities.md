# B2B E-Commerce Entity Development Prompt

## System Overview
You are developing a B2B shopping cart where businesses can sell products to other companies. The system focuses on creating and managing orders between organizations, with a private relationship between the seller and clients. The system does not track payments, shipping, tax, or warehouse inventory.

## Existing Entities
The system already has the following entities implemented:

1. **Order** - Represents an order placed by a client organization
2. **OrderItem** - Represents individual items within an order
3. **Organization** - Represents business entities using the system
4. **Branch** - Represents individual branches of an organization
5. **Product** - Represents products available for purchase
6. **Category** - Represents product categories

## Required New Entities
Develop the following additional entities with appropriate relationships:

1. **BranchUser** - Connect branches with external user system
   - Should reference external users by cloudUserId
   - Each BranchUser belongs to exactly one Organization
   - Implements a many-to-many relationship where each user can belong to multiple branches (within the same organization)
   - Implements a many-to-many relationship where each user can have multiple cloud roles
   - Requires appropriate join tables to manage these many-to-many relationships

2. **CloudRole** - Reference roles from the external cloud system
   - Store cloud role identifiers and names
   - Do not implement local role management
   - Will be linked to external cloud user system
   - Serves only as a reference to permissions defined in the cloud system

3. **Country** - Store standardized country information
   - Use ISO 3166-1 alpha-2 for country codes (e.g., US, CA, GB)
   - Include country name and code fields
   - Serve as reference data for addresses

4. **State/Province** - Store standardized state/province information
   - Use ISO 3166-2 format for state/province codes (e.g., US-NY, CA-ON)
   - Include name, code, and relationship to country
   - Serve as reference data for addresses

5. **City** - Store city information
   - Include city name and relationship to state/province
   - May include postal code patterns if applicable
   - Serve as reference data for addresses

6. **Address** - Store location information
   - Should include references to Country, State/Province, and City entities
   - Include street address and zipcode fields
   - Can be associated with both Organizations and Branches
   - Include validation to ensure proper ISO format compliance

7. **PriceList** - Manage different pricing structures
   - Allow for organization-specific pricing
   - Support default and custom price lists

8. **PriceListItem** - Individual product pricing within a price list
   - Connect products to specific prices
   - Support organization-specific pricing

9. **Cart** - Temporary storage before order creation
   - Associate with specific branch users
   - Track items added to cart before checkout

10. **CartItem** - Individual products in a cart
   - Track quantity of each product
   - Associate with specific cart

## Key Requirements
1. Implement proper JPA annotations and relationships
2. Follow existing patterns for bidirectional relationships
3. Use the AuditModel as the base class for all entities
4. Include appropriate validation annotations
5. Implement proper lazy loading where appropriate
6. Override equals and hashCode methods following the pattern in existing entities
7. Implementation of helper methods for relationship management

## Integration Notes
- User authentication and core user data are managed by an external microservice
- The system only stores references to users via cloudUserId
- Each branch can have multiple users
- Users can be assigned different roles within the system

## Non-Requirements
Do not implement entities for:
- Quote/RFQ processes
- Return/Refund management
- Contract/Agreement management
- Tax calculation
- Warehouse/Inventory management
- Payment processing
- Shipping/Delivery tracking

## Entity Structure Pattern
Follow the existing pattern for creating new entities:

```java
package app.quantun.eb2c.model.entity.bussines;

import app.quantun.eb2c.model.entity.AuditModel;
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import lombok.*;
import org.hibernate.proxy.HibernateProxy;

import java.util.Objects;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "entity_name")
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class EntityName extends AuditModel<String> {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Fields, relationships, and methods
    
    @Override
    public final boolean equals(Object o) {
        if (this == o) return true;
        if (o == null) return false;
        Class<?> oEffectiveClass = o instanceof HibernateProxy ? ((HibernateProxy) o).getHibernateLazyInitializer().getPersistentClass() : o.getClass();
        Class<?> thisEffectiveClass = this instanceof HibernateProxy ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass() : this.getClass();
        if (thisEffectiveClass != oEffectiveClass) return false;
        EntityName entity = (EntityName) o;
        return getId() != null && Objects.equals(getId(), entity.getId());
    }

    @Override
    public final int hashCode() {
        return this instanceof HibernateProxy ? ((HibernateProxy) this).getHibernateLazyInitializer().getPersistentClass().hashCode() : getClass().hashCode();
    }
}
```

Develop these entities ensuring they integrate well with the existing system architecture and follow the established patterns.