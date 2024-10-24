# Entity Framework Core - ORM Overview

## What is an ORM?

An Object-Relational Mapper (ORM) is a tool that enables mapping between:
- Application objects (Desktop, Mobile, or Web applications)
- Database relations (tables)

```mermaid
graph LR
    A[Application Objects] <--> B[ORM] <--> C[Database Relations]
    style B fill:#f9f,stroke:#333,stroke-width:4px
```

### Key Concepts
- **Object**: Instance of a class in your application
- **Relation**: Table in your database schema
- **Mapping**: Converting between objects and relations bidirectionally

### Benefits
- Focus on one development approach (either object-oriented or database-first)
- Abstract away direct database interactions
- Reduce database vendor lock-in
- Write database operations in your application's native language

## Entity Framework Core

Entity Framework Core is Microsoft's official ORM for .NET applications. For .NET 6 applications, we use Entity Framework Core 6.

### Mapping Approaches

There are two main approaches to mapping in EF Core:

| Approach | Direction | Description | Common Use Case |
|----------|-----------|-------------|----------------|
| Code First | Classes → Database | Generate database tables and views from your code | New projects, domain-driven design |
| Database First | Database → Classes | Generate classes from existing database schema | Legacy database integration |

#### Why Code First is Popular
- Easier development workflow
- Better control over the development cycle
- Changes start in the code
- Version control friendly
- Supports agile development practices

### Working with Data

Instead of writing raw SQL, EF Core allows you to:
1. Write LINQ queries (LINQ to Entities - L2E)
2. Query against remote sequences (tables, views, functions)
3. Operate on object models that represent database rows

```mermaid
graph TD
    A[Application Code] -->|LINQ| B[Entity Framework Core]
    B -->|SQL| C[Database]
    style B fill:#f9f,stroke:#333,stroke-width:4px
```

### Object-Database Mapping Reference

| Application Concept | Database Equivalent |
|--------------------|---------------------|
| Object Model | Database Table |
| Object Instance | Table Row |
| Property | Column |
| LINQ Query | SQL Query |

### Key Operations
All database operations are performed through C# code:
- Create (Insert)
- Read (Select)
- Update
- Delete

These operations are written in LINQ and automatically converted to appropriate SQL commands by EF Core.


# Entity Framework Core - Inheritance Mapping Strategies

## Understanding Inheritance Mapping

When working with inheritance in Entity Framework Core, the way classes are mapped to database tables significantly impacts application performance and maintainability.

## Class Structure Example

Consider an inheritance hierarchy with employees:

```mermaid
classDiagram
    Employee <|-- FullTimeEmployee
    Employee <|-- PartTimeEmployee
    
    class Employee {
        +ID: int
        +Name: string
        +Age: int
        +Address: string
    }
    
    class FullTimeEmployee {
        +StartDate: DateTime
        +Salary: decimal
    }
    
    class PartTimeEmployee {
        +HourRate: decimal
        +CountOfHours: int
    }
```

## Table Per Class (TPC) Strategy

### Overview
TPC is the default inheritance mapping strategy in EF Core. With this approach:
- Each concrete class in the inheritance hierarchy gets its own database table
- Each table contains columns for all properties (inherited and declared)

### Database Schema with TPC
```mermaid
erDiagram
    Employee {
        int ID PK
        string Name
        int Age
        string Address
    }
    
    FullTimeEmp {
        int FullEmpID PK
        DateTime StartDate
        decimal Salary
    }
    
    PartTimeEmp {
        int PartEmpID PK
        decimal HourRate
        int CountOfHours
    }
```

### Disadvantages of TPC

1. **Data Redundancy**
   - Common properties are stored in multiple tables
   - Increases storage requirements
   - Complicates data maintenance

2. **Performance Impact**
   - CRUD operations require multiple table access
   - Creating a new employee requires inserts into multiple tables
   - Queries often need JOIN operations

3. **Operational Complexity**
   - Updates and deletes must maintain consistency across tables
   - More complex query generation
   - Higher chance of data inconsistency

### When to Avoid TPC
- In scenarios with deep inheritance hierarchies
- When performance is a critical requirement
- When data consistency is paramount
- In systems with high transaction volumes

### Code First Considerations
When working Code First:
- TPC is applied by default
- Consider carefully whether this is the best approach for your use case
- Alternative mapping strategies might be more appropriate

## Best Practices
1. Evaluate inheritance mapping strategy early in design
2. Consider the trade-offs between storage, performance, and complexity
3. Don't automatically accept the default TPC strategy
4. Test performance with representative data volumes


# Entity Framework Core - Advanced Inheritance Mapping Strategies

## Table Per Hierarchy (TPH)

TPH is an alternative to TPC that stores the entire inheritance hierarchy in a single table.

### Single Table Structure
```mermaid
erDiagram
    Employee {
        int ID PK
        string Name
        int Age
        string Address
        decimal Salary
        datetime StartDate
        decimal HourRate
        int CountOfHours
        string Discriminator
    }
```

### Key Characteristics
1. **Discriminator Column**
   - Automatically added by EF Core
   - Acts as an enum to identify row type (FullTime/PartTime)
   - Helps distinguish between different types of employees

2. **Advantages**
   - Single table for all operations
   - Simpler querying
   - Better performance for CRUD operations
   - No need for table joins

3. **Disadvantages**
   - NULL values for non-applicable fields
   - Less efficient storage utilization
   - Potential for wide tables with many nullable columns

## Table Per Concrete Class (TPCC)

Introduced in 2019, TPCC maps only concrete classes to database tables.

### Database Schema
```mermaid
erDiagram
    FullTimeEmployee {
        int ID PK
        string Name
        int Age
        string Address
        decimal Salary
        datetime StartDate
    }
    
    PartTimeEmployee {
        int ID PK
        string Name
        int Age
        string Address
        decimal HourRate
        int CountOfHours
    }
```

### Key Features
1. **Concrete Classes Only**
   - Only maps classes with actual business representation
   - Base/abstract classes (like Employee) are not mapped
   - Improved storage efficiency

2. **Advantages**
   - No nullable columns
   - Clean database design
   - Better storage efficiency
   - Single table operations for CRUD
   - Maps closely to business entities

3. **Use Cases**
   - When abstract classes are purely for code organization
   - When storage efficiency is important
   - When clear separation between types is needed

## Choosing the Right Strategy

### Comparison Matrix

| Feature | TPC | TPH | TPCC |
|---------|-----|-----|------|
| Storage Efficiency | Poor | Medium | Good |
| Query Performance | Poor | Good | Good |
| NULL Values | No | Yes | No |
| Table Count | Many | One | Few |
| Maintenance | Complex | Simple | Medium |

### Database Vendor Support
- SQL Server
- MySQL
- PostgreSQL
- Other vendors supported by EF Core

### Best Practices
1. Consider TPCC for new projects
2. Use TPH when storage isn't a primary concern
3. Avoid TPC unless specifically required
4. Test performance with realistic data volumes
5. Consider the specific requirements of your chosen database vendor




# .NET Data Access Technologies Comparison

## Evolution of Data Access in .NET

```mermaid
graph TD
    A[ADO.NET] --> B[Entity Framework]
    A --> C[Dapper]
    B --> D[Entity Framework Core]
    style D fill:#f9f,stroke:#333,stroke-width:4px
```

## Technology Overview

### 1. Entity Framework Core
- Primary full-featured ORM for .NET
- Built on top of ADO.NET
- Features:
  - LINQ support
  - Multiple database providers
  - Rich mapping capabilities
  - Migration management
  - Change tracking
  - Lazy loading

### 2. Dapper
- Lightweight "micro-ORM"
- Developed by StackOverflow team
- Key characteristics:
  - Direct SQL writing
  - Faster data retrieval in specific scenarios
  - Minimal overhead
  - Simple object mapping

### 3. ADO.NET (Legacy)
- Base data access layer
- Direct database interaction
- Used internally by EF Core
- Maximum performance but more complex code

## Performance Comparison

| Feature | EF Core | Dapper | ADO.NET |
|---------|---------|---------|----------|
| Query Speed | Good | Excellent | Excellent |
| Development Speed | Excellent | Good | Fair |
| Code Complexity | Low | Medium | High |
| Learning Curve | Steep | Moderate | Steep |
| Maintenance | Easy | Moderate | Complex |

### Benchmark Considerations
- Use benchmark tools to measure query efficiency
- Consider scenarios:
  - Data retrieval volume
  - Query complexity
  - Update frequency

## Modern Best Practices

### 1. Hybrid Approach
```mermaid
graph LR
    A[Application] --> B{Data Access Layer}
    B --> C[EF Core<br/>Complex Operations]
    B --> D[Dapper<br/>Performance-Critical Queries]
```

### 2. When to Use Each Technology

| Technology | Best Used For |
|------------|--------------|
| EF Core | - Complex domain models<br/>- CRUD operations<br/>- Database agnostic code |
| Dapper | - Performance-critical reads<br/>- Simple queries<br/>- Direct SQL control |
| ADO.NET | - Legacy systems<br/>- Extremely performance-critical operations |

### 3. Package Management
```plaintext
Required NuGet Packages:
- Microsoft.EntityFrameworkCore
- Dapper
```

## Trade-offs Analysis

### Entity Framework Core
✅ Advantages:
- Rich feature set
- Database independence
- LINQ support
- Less code
- Modern development experience

⚠️ Considerations:
- Performance overhead
- Learning curve
- Memory usage

### Dapper
✅ Advantages:
- Better performance for reads
- Lightweight
- Simple to use
- Direct SQL control

⚠️ Considerations:
- Manual SQL writing
- Less abstraction
- More code for complex operations

## Development Recommendations

1. **Start with EF Core**
   - Use for majority of data access
   - Leverage built-in features
   - Focus on clean domain model

2. **Add Dapper When Needed**
   - Identify performance-critical paths
   - Use for read-heavy operations
   - Benchmark to verify improvements

3. **Avoid ADO.NET**
   - Unless specifically required
   - Consider only for legacy maintenance
   - Migration path to modern ORMs





# Entity Framework Core - Main Concepts Summary

## 1. Mapping (Code Design Tools)

Entity Framework Core provides two primary approaches for mapping between your code and database:

```mermaid
graph TD
    A[Mapping Approaches] --> B[Database First]
    A --> C[Code First]
    B --> D[Generate Classes from DB]
    C --> E[Generate DB from Classes]
    style A fill:#f9f,stroke:#333,stroke-width:4px
```

### 1.1 Database First Approach
- Starts with existing database
- Generates entity classes from database schema
- Best for:
  - Legacy database integration
  - Complex existing databases
  - Database-driven development

### 1.2 Code First Approach
- Starts with C# classes (entities)
- Generates database tables and views
- Preferred for:
  - New projects
  - Domain-driven design
  - Greater control over data model

## 2. L2E (LINQ to Entity)

LINQ to Entity provides a way to query your object model using LINQ instead of direct SQL.

### 2.1 Comparison with Traditional Approaches

| Feature | ADO.NET | Entity Framework Core |
|---------|---------|---------------------|
| Query Language | Stored Procedures | LINQ (L2E) |
| SQL Generation | Manual | Automatic |
| Query Optimization | Manual | Built-in |
| Database Access | Direct | Abstracted |
| Code Complexity | Higher | Lower |

### 2.2 CRUD Operations

```mermaid
graph LR
    A[Application Code] -->|LINQ| B[EF Core]
    B -->|Optimized SQL| C[Database]
    D[ADO.NET] -->|Stored Procedures| C
```

#### EF Core Approach
- Write LINQ queries in C#
- EF Core converts to optimized SQL
- Automatic query optimization
- Database-agnostic code
- Simplified CRUD operations

#### ADO.NET Approach
- Use stored procedures
- Direct database interaction
- Manual SQL optimization
- Database-specific code
- More verbose CRUD operations

## Best Practices

### Design Considerations
1. Choose appropriate mapping strategy
   - Consider existing infrastructure
   - Evaluate team expertise
   - Assess project requirements

2. Query Optimization
   - Use appropriate LINQ methods
   - Monitor query performance
   - Consider Dapper for performance-critical reads

### Implementation Guidelines
1. Code First:
   - Use meaningful entity names
   - Define relationships clearly
   - Implement proper inheritance strategy (TPH, TPC, TPCC)

2. Database First:
   - Maintain clean database schema
   - Use consistent naming conventions
   - Document complex relationships

3. LINQ to Entity:
   - Write clean, readable LINQ queries
   - Avoid N+1 query problems
   - Use appropriate loading strategies (eager, lazy, explicit)

## Key Benefits
- ✅ Simplified data access
- ✅ Reduced development time
- ✅ Database independence
- ✅ Automatic SQL optimization
- ✅ Modern development experience
