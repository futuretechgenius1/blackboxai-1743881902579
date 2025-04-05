# Excel Comparator - Implementation Details

## Architecture Overview

The Excel Comparator follows a layered architecture pattern:

```
Presentation Layer (Thymeleaf Templates)
            ↓
Controller Layer (REST endpoints)
            ↓
Service Layer (Business Logic)
            ↓
Repository Layer (Data Access)
            ↓
Database Layer (MySQL)
```

## Component Details

### 1. Domain Models

#### ComparisonField
- Represents a field to be compared between Excel files
- Properties:
  - id: Unique identifier
  - fieldName: Internal name
  - columnIndex: Excel column position (0-based)
  - displayLabel: User-friendly name
  - isKey: Whether field is used for row matching
  - active: Field status
  - validationRules: Associated validation rules

#### ValidationRule
- Defines validation criteria for fields
- Types:
  - REGEX: Pattern matching
  - RANGE: Numeric boundaries
  - NOT_NULL: Required field
  - CUSTOM: Custom validation logic
- Properties:
  - id: Unique identifier
  - comparisonField: Associated field
  - ruleType: Type of validation
  - ruleExpression: Validation criteria
  - active: Rule status

#### ComparisonResult
- Stores comparison outcome
- Status types:
  - MATCH: Values are identical
  - MISMATCH: Values differ
  - MISSING_IN_BASE: Row absent in base file
  - MISSING_IN_TARGET: Row absent in target file
  - VALIDATION_ERROR: Value fails validation

### 2. Services

#### UploadService
- Handles file upload operations
- Features:
  - File type validation (.xlsx, .xls)
  - Size limit enforcement (10MB)
  - Temporary storage management
  - Automatic cleanup
- Key Methods:
  ```java
  String storeFile(MultipartFile file)
  void deleteFile(String filePath)
  boolean isValidExcelFile(MultipartFile file)
  ```

#### ComparisonService
- Core comparison logic
- Operations:
  - Excel file reading using Apache POI
  - Row matching using key fields
  - Value comparison
  - Validation rule application
  - Result generation
- Key Methods:
  ```java
  List<ComparisonResult> compareExcelFiles(String baseFilePath, String targetFilePath)
  Map<String, Row> mapRowsByKeys(Sheet sheet, List<ComparisonField> keyFields)
  void compareRows(Map<String, Row> baseRows, Map<String, Row> targetRows)
  ```

#### ReportService
- Generates comparison reports
- Features:
  - Excel file generation
  - Color coding:
    - Green: Matches
    - Red: Mismatches
    - Yellow: Missing data
  - Summary statistics
  - Detailed field comparisons
- Key Methods:
  ```java
  String generateReport(List<ComparisonResult> results, String outputDir)
  void createSummarySheet(Workbook workbook, List<ComparisonResult> results)
  ```

#### ConfigurationService
- Manages comparison configuration
- Features:
  - Field CRUD operations
  - Validation rule management
  - Active/inactive status control
  - Configuration validation
- Key Methods:
  ```java
  ComparisonField createComparisonField(ComparisonField field)
  ValidationRule createValidationRule(ValidationRule rule)
  void validateComparisonField(ComparisonField field)
  void validateValidationRule(ValidationRule rule)
  ```

### 3. Controllers

#### ExcelController
- Endpoints:
  ```
  GET  /excel              - Show upload form
  POST /excel/upload       - Process files
  GET  /excel/download/{*} - Download report
  ```

#### ConfigController
- Endpoints:
  ```
  # Field Management
  GET  /config                    - Dashboard
  GET  /config/fields/new         - New field form
  POST /config/fields             - Create field
  GET  /config/fields/{id}/edit   - Edit field form
  POST /config/fields/{id}        - Update field
  POST /config/fields/{id}/delete - Delete field

  # Rule Management
  GET  /config/fields/{id}/rules      - List rules
  GET  /config/fields/{id}/rules/new  - New rule form
  POST /config/fields/{id}/rules      - Create rule
  GET  /config/rules/{id}/edit        - Edit rule form
  POST /config/rules/{id}             - Update rule
  POST /config/rules/{id}/delete      - Delete rule
  ```

### 4. Database Schema

#### comparison_fields
```sql
CREATE TABLE comparison_fields (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    field_name VARCHAR(255) NOT NULL,
    column_index INT NOT NULL,
    display_label VARCHAR(255) NOT NULL,
    is_key BOOLEAN DEFAULT FALSE,
    active BOOLEAN DEFAULT TRUE,
    description VARCHAR(500)
);
```

#### validation_rules
```sql
CREATE TABLE validation_rules (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    comparison_field_id BIGINT NOT NULL,
    rule_type VARCHAR(50) NOT NULL,
    rule_expression VARCHAR(500) NOT NULL,
    description VARCHAR(500),
    active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (comparison_field_id) 
        REFERENCES comparison_fields(id) 
        ON DELETE CASCADE
);
```

### 5. User Interface

#### Templates
- layout.html: Base template with navigation
- upload.html: File upload interface
- result.html: Comparison results display
- config/*.html: Configuration management screens

#### Features
- Responsive design using Bootstrap
- Interactive forms with client-side validation
- Real-time feedback
- Progress indicators
- Error handling with user-friendly messages

### 6. Excel Processing

#### File Reading
- Uses Apache POI
- Supports both .xlsx and .xls formats
- Handles multiple data types:
  - Text
  - Numbers
  - Dates
  - Formulas

#### Comparison Logic
1. Read both Excel files
2. Map rows using key fields
3. Compare corresponding fields
4. Apply validation rules
5. Generate comparison results
6. Create detailed report

### 7. Validation System

#### Rule Types
1. REGEX
   - Pattern matching for text
   - Example: `^[A-Z]{2}\d{4}$`

2. RANGE
   - Numeric boundary checking
   - Format: `min,max`
   - Example: `0,100`

3. NOT_NULL
   - Required field validation
   - Value: `true`

4. CUSTOM
   - Extensible validation logic
   - Implementation-specific

#### Validation Process
1. Load active rules for field
2. Apply rules in sequence
3. Stop on first failure
4. Record validation errors

### 8. Error Handling

#### Types
- File validation errors
- Processing errors
- Configuration errors
- Database errors

#### Implementation
- Global exception handler
- User-friendly error messages
- Detailed logging
- Error recovery suggestions

### 9. Security Considerations

- Input validation
- File type verification
- Size limitations
- SQL injection prevention
- XSS protection

### 10. Performance Optimization

- Database indexing
- Lazy loading of relationships
- Efficient Excel processing
- Temporary file cleanup
- Connection pooling

## Development Guidelines

### Adding New Features

1. Create/update domain models
2. Implement repository interfaces
3. Add service layer logic
4. Create controller endpoints
5. Design UI templates
6. Add validation rules
7. Update documentation

### Best Practices

1. Follow SOLID principles
2. Write unit tests
3. Use meaningful naming
4. Add proper documentation
5. Handle errors gracefully
6. Implement logging
7. Consider security implications

### Deployment Considerations

1. Database setup
2. Environment configuration
3. File storage configuration
4. Memory requirements
5. Monitoring setup
6. Backup strategy

## Future Enhancements

1. Additional validation types
2. Custom comparison logic
3. Report customization
4. Batch processing
5. API integration
6. User authentication
7. Role-based access control