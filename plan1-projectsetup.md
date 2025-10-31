I have created the following plan after thorough exploration and analysis of the codebase. Follow the below plan verbatim. Trust the files and references. Do not re-verify what's written in the plan. Explore only when absolutely necessary. First implement all the proposed file changes and then I'll review all the changes together at the end.

### Observations

The repository is currently empty except for README.md and LICENSE files. This is a greenfield Maven project that requires complete setup from scratch. The task focuses on establishing the foundational project structure, dependencies, logging configuration, and test resources needed for a Java CLI PDF form-filler application. The subsequent phases will build upon this foundation with TDD implementation of core components.

### Approach

Create a standard Maven project structure with proper directory hierarchy, configure `pom.xml` with all required dependencies (PDFBox 3.0.6, Picocli 4.7.7, SLF4J 2.0.17, Logback 1.5.20, Lombok 1.18.42, JUnit 5, AssertJ), set up Logback configuration for both production and test environments, acquire/create sample PDF forms with various field types for testing, and establish the main package structure following Java conventions.

### Reasoning

I listed the repository contents and found it to be essentially empty. I then searched the web for the latest stable versions of all required dependencies to ensure the project uses current, secure versions. I also researched tools for creating sample PDF forms with fillable fields for testing purposes.

## Proposed File Changes

### pom.xml(NEW)

Create Maven Project Object Model (POM) file with:

**Project Coordinates:**
- groupId: `com.example` (or appropriate organization identifier)
- artifactId: `pdf-form-filler`
- version: `1.0.0-SNAPSHOT`
- packaging: `jar`
- name: `PDF Form Filler CLI`
- description: `Command-line application for filling PDF forms from property files`

**Properties:**
- `maven.compiler.source`: `17` (or `11` minimum for modern Java)
- `maven.compiler.target`: `17` (or `11`)
- `project.build.sourceEncoding`: `UTF-8`
- `project.reporting.outputEncoding`: `UTF-8`
- Dependency versions as properties for easy maintenance:
  - `pdfbox.version`: `3.0.6`
  - `picocli.version`: `4.7.7`
  - `slf4j.version`: `2.0.17`
  - `logback.version`: `1.5.20`
  - `lombok.version`: `1.18.42`
  - `junit.version`: `5.11.3` (latest JUnit 5)
  - `assertj.version`: `3.26.3` (latest AssertJ)

**Dependencies (compile scope):**
- `org.apache.pdfbox:pdfbox:${pdfbox.version}` - for PDF manipulation and form filling
- `info.picocli:picocli:${picocli.version}` - for CLI argument parsing with annotations
- `org.slf4j:slf4j-api:${slf4j.version}` - logging facade
- `ch.qos.logback:logback-classic:${logback.version}` - logging implementation (includes logback-core transitively)
- `org.projectlombok:lombok:${lombok.version}` with `<scope>provided</scope>` - for code generation (@Slf4j, @Data, etc.)

**Dependencies (test scope):**
- `org.junit.jupiter:junit-jupiter:${junit.version}` - JUnit 5 testing framework
- `org.assertj:assertj-core:${assertj.version}` - fluent assertions for better test readability

**Build Configuration:**
- `maven-compiler-plugin` (version 3.13.0 or latest) with:
  - Annotation processor path for Lombok to enable compile-time code generation
  - Configuration: `<annotationProcessorPaths>` containing Lombok dependency
- `maven-surefire-plugin` (version 3.5.2 or latest) for running JUnit 5 tests
- `maven-jar-plugin` with manifest configuration:
  - `Main-Class`: `com.example.pdfformfiller.PdfFormFillerCli` (main entry point)
  - `addClasspath`: `true`
- Optional: `maven-shade-plugin` or `maven-assembly-plugin` for creating an executable uber-JAR with all dependencies bundled (useful for CLI distribution)

### src(NEW)

Create root source directory following Maven standard directory layout. This directory will contain both main application code and test code subdirectories.

### src/main(NEW)

Create main source directory for production code and resources.

### src/main/java(NEW)

Create directory for Java source files (production code).

### src/main/java/com(NEW)

Create top-level package directory following Java package naming conventions.

### src/main/java/com/example(NEW)

Create organization-level package directory.

### src/main/java/com/example/pdfformfiller(NEW)

Create main application package directory. This will contain all production Java classes including:
- CLI command class (Picocli-based)
- Core components: `PropertyParser`, `PdfFormProcessor`, `ReportGenerator`
- File I/O adapter layer
- Data model classes (e.g., `ReportData`)
- Utility classes if needed

### src/main/resources(NEW)

Create directory for application resources (configuration files, templates, etc.).

### src/main/resources/logback.xml(NEW)

Create Logback configuration for production/runtime logging with:

**Appenders:**
- `CONSOLE` appender:
  - Target: `System.out`
  - Pattern: `%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n`
  - Threshold: `INFO` (or `DEBUG` for verbose mode)
- `FILE` appender:
  - File path: configurable via system property `log.file` with default fallback (e.g., `${log.file:-pdf-form-filler.log}`)
  - Rolling policy: `TimeBasedRollingPolicy` with daily rotation
  - File name pattern: `pdf-form-filler-%d{yyyy-MM-dd}.log`
  - Max history: 30 days
  - Pattern: same as console for consistency
  - Threshold: `DEBUG` to capture detailed information for troubleshooting

**Root Logger:**
- Level: `INFO`
- Appender refs: both `CONSOLE` and `FILE`

**Package-specific Loggers (optional):**
- `com.example.pdfformfiller`: `DEBUG` level to capture detailed application flow
- `org.apache.pdfbox`: `WARN` level to reduce noise from PDFBox internals

**Notes:**
- The file path should be configurable via CLI argument (handled by the CLI layer setting system property)
- Use immediate flush for file appender to ensure logs are written even if application crashes
- Consider adding `<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">` for explicit encoding configuration

### src/test(NEW)

Create test source directory for test code and test resources.

### src/test/java(NEW)

Create directory for Java test source files.

### src/test/java/com(NEW)

Create top-level package directory for tests (mirrors production structure).

### src/test/java/com/example(NEW)

Create organization-level package directory for tests.

### src/test/java/com/example/pdfformfiller(NEW)

Create main test package directory. This will contain all test classes including:
- Unit tests for `PropertyParser` (testing escape sequence handling: `\=` → `=`, `\\` → `\`)
- Unit tests for `PdfFormProcessor` (testing PDF form filling with various field types)
- Unit tests for `ReportGenerator` (testing report generation)
- Integration tests for complete CLI flow
- Test utilities and helper classes

### src/test/resources(NEW)

Create directory for test resources (sample PDFs, test property files, expected outputs, etc.).

### src/test/resources/logback-test.xml(NEW)

Create Logback configuration specifically for test execution with:

**Appenders:**
- `CONSOLE` appender only (no file appender needed during tests):
  - Target: `System.out`
  - Pattern: `%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n` (shorter timestamp for test readability)
  - Threshold: `DEBUG` for detailed test diagnostics

**Root Logger:**
- Level: `DEBUG` (more verbose than production to catch test issues)
- Appender ref: `CONSOLE`

**Package-specific Loggers:**
- `com.example.pdfformfiller`: `DEBUG` level
- `org.apache.pdfbox`: `WARN` level to reduce PDFBox noise during tests
- `org.junit`: `INFO` level

**Notes:**
- This configuration takes precedence over `logback.xml` during test execution
- Keep it simple and console-only since tests run in controlled environments
- Consider adding `<turboFilter>` if specific test classes need different log levels

### src/test/resources/pdfs(NEW)

Create directory for sample PDF form templates used in testing. This directory will contain multiple test PDFs with different characteristics.

### src/test/resources/pdfs/sample-form-simple.pdf(NEW)

Acquire or create a simple PDF form with basic text fields for initial testing:

**Recommended Fields:**
- `firstName` (text field)
- `lastName` (text field)
- `email` (text field)
- `phone` (text field)
- `address` (text field, possibly multiline)

**Creation Options:**
1. **PDFescape Online** (fastest, browser-based, free):
   - Visit pdfescape.com
   - Create new PDF document
   - Insert → Form Field → Text for each field
   - Set field names in Properties panel
   - Download as `sample-form-simple.pdf`

2. **LibreOffice Writer** (offline, free):
   - Create document with form controls
   - Insert → Form Controls → Text Box for each field
   - Right-click → Control Properties → set Name property
   - File → Export as PDF → enable "Create PDF Form"

3. **Adobe Acrobat** (if available):
   - Create form using Prepare Form tool
   - Add text fields with appropriate names

**Testing Purpose:**
- Verify basic text field filling functionality
- Test property-to-field name mapping
- Validate that filled PDF can be read back with PDFBox `PDDocument.load()` and `PDAcroForm.getField()`

**Note:** Ensure field names match exactly what will be used in test property files (case-sensitive)

### src/test/resources/pdfs/sample-form-complex.pdf(NEW)

Acquire or create a more complex PDF form with various field types for comprehensive testing:

**Recommended Fields:**
- Text fields: `name`, `email`, `comments` (multiline)
- Checkbox fields: `subscribe`, `agreeToTerms`, `newsletter`
- Radio button groups: `gender` (options: Male, Female, Other), `employmentType` (options: FullTime, PartTime, Contractor)
- Dropdown/Combo fields: `department` (options: HR, Engineering, Marketing, Sales, Finance), `country`
- Date field: `startDate` (if supported by creation tool)

**Creation Options:**
1. **PDFescape Online**:
   - Create new PDF
   - Insert various field types: Text, Checkbox, Radio, Dropdown
   - Configure radio button groups with same group name but different export values
   - Set meaningful field names

2. **LibreOffice Writer**:
   - Use Form Controls toolbar
   - Add Text Box, Check Box, Option Button (radio), Combo Box
   - Group radio buttons by setting same name
   - Export as PDF form

**Testing Purpose:**
- Verify handling of different field types (text, checkbox, radio, dropdown)
- Test radio button group selection (only one option selected)
- Test checkbox state (checked/unchecked based on property values like "true"/"false" or "yes"/"no")
- Validate multiline text field handling
- Test dropdown value selection

**Note:** Document the exact field names and expected values in a companion README or comment in test classes

### src/test/resources/pdfs/sample-form-special-chars.pdf(NEW)

Acquire or create a PDF form with field names containing special characters to test edge cases:

**Recommended Fields with Special Names:**
- `user.name` (dot in name)
- `user_email` (underscore)
- `address-line1` (hyphen)
- `field with spaces` (spaces, if supported)
- `field=value` (equals sign in field name - tests property escaping)
- `path\\to\\file` (backslashes - tests property escaping)

**Creation:**
- Use any of the tools mentioned above (PDFescape, LibreOffice)
- Carefully set field names to include special characters
- Some tools may restrict certain characters; document limitations

**Testing Purpose:**
- Verify property file escape sequence handling: `\=` → `=`, `\\` → `\`
- Test that field names with special characters are correctly matched
- Validate property key parsing with escaped characters (e.g., `field\=value=some text` should map to field named `field=value`)
- Ensure robust handling of various naming conventions

**Note:** This PDF is specifically for testing the property parser's escape sequence functionality as specified in requirements

### src/test/resources/pdfs/README.md(NEW)

Create documentation file describing the test PDF forms:

**Content:**
- Overview of each PDF file and its purpose
- List of field names in each PDF with their types (text, checkbox, radio, dropdown)
- Expected values for testing (e.g., for radio buttons: which export values to use)
- Instructions for regenerating PDFs if needed
- Tools used to create the PDFs
- Any known limitations or quirks of the PDF forms

**Example Structure:**
```
# Test PDF Forms

## sample-form-simple.pdf
- **Purpose:** Basic text field testing
- **Fields:**
  - firstName (text)
  - lastName (text)
  - email (text)
  - phone (text)
  - address (text, multiline)

## sample-form-complex.pdf
- **Purpose:** Multiple field types
- **Fields:**
  - name (text)
  - email (text)
  - subscribe (checkbox, values: "Yes"/"Off")
  - gender (radio group, options: Male/Female/Other)
  - department (dropdown, options: HR/Engineering/Marketing/Sales/Finance)
  - comments (text, multiline)

## sample-form-special-chars.pdf
- **Purpose:** Edge case testing for special characters in field names
- **Fields:**
  - user.name (text)
  - field=value (text) - requires escaping in properties: field\=value=...
  - path\\to\\file (text) - requires escaping: path\\\\to\\\\file=...
```

**Note:** This documentation will be invaluable for writing tests and understanding test failures

### src/test/resources/properties(NEW)

Create directory for sample property files used in testing. These will contain test data for filling the PDF forms, including edge cases like escape sequences.

### .gitignore(MODIFY)

Create Git ignore file for Maven Java project with:

**Maven-specific:**
- `target/` - build output directory
- `pom.xml.tag`
- `pom.xml.releaseBackup`
- `pom.xml.versionsBackup`
- `pom.xml.next`
- `release.properties`
- `dependency-reduced-pom.xml`
- `buildNumber.properties`
- `.mvn/timing.properties`
- `.mvn/wrapper/maven-wrapper.jar`

**IDE-specific:**
- `.idea/` - IntelliJ IDEA
- `*.iml` - IntelliJ module files
- `.vscode/` - Visual Studio Code
- `.settings/` - Eclipse
- `.project` - Eclipse
- `.classpath` - Eclipse
- `*.swp`, `*.swo` - Vim
- `.DS_Store` - macOS

**Application-specific:**
- `*.log` - log files generated by the application
- `*.pdf` - generated PDF files (except test resources)
- `*.properties` - generated property files (except test resources)
- `reports/` - generated report directory if used

**Lombok-specific:**
- `.apt_generated/` - annotation processor output

**Note:** Ensure test resources (`src/test/resources/`) are NOT ignored as they contain essential test PDFs and property files

### README.md(MODIFY)

Enhance the README with comprehensive project documentation:

**Add Sections:**

1. **Project Description:**
   - Brief overview: Java CLI application for filling PDF forms from property files
   - Key features: stream-based architecture, detailed logging, processing reports
   - Technology stack: Java 17, Maven, PDFBox, Picocli, SLF4J/Logback, Lombok

2. **Requirements:**
   - Java 17 or higher
   - Maven 3.6+ for building

3. **Building the Project:**
   - `mvn clean install` - compile and run tests
   - `mvn package` - create executable JAR
   - Location of built JAR: `target/pdf-form-filler-1.0.0-SNAPSHOT.jar`

4. **Usage (placeholder for future implementation):**
   - Command syntax: `java -jar pdf-form-filler.jar --input <template.pdf> --properties <data.properties> --output <filled.pdf> [--log <logfile.log>] [--report <report.txt>]`
   - Flag descriptions
   - Example invocation

5. **Property File Format:**
   - Standard Java properties format
   - Escape sequences: `\=` for literal equals, `\\` for literal backslash
   - Examples:
     - `firstName=Max`
     - `field\=name=value` (field name contains `=`)
     - `path\\to\\file=C:\\temp` (backslashes in value)

6. **Development:**
   - Project structure overview
   - Running tests: `mvn test`
   - Test resources location: `src/test/resources/pdfs/`
   - Logging configuration: `src/main/resources/logback.xml`

7. **Architecture (high-level):**
   - Stream-based core components for testability
   - Separation of concerns: parsing, processing, reporting, CLI
   - TDD approach with comprehensive unit tests

8. **License:**
   - Reference to LICENSE file

**Note:** Keep the existing content and expand it with these sections. Use clear markdown formatting with headers, code blocks, and lists for readability.