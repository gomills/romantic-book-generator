# Romantic Personalized Book Generator

A FastAPI application that generates personalized, style and content consistent relationship books using OpenAI's API for text and image generation.

![Sample Image](https://raw.githubusercontent.com/gomills/romantic-book-generator/refs/heads/main/book_sample.png?raw=true)
## Overview

This service creates custom PDF books based on user-provided information about their couple-relationship. The system:

1. Collects user input via a web form
2. Validates and processes the input
3. Generates personalized text content and images using GPT models
4. Compiles everything into a professionally formatted PDF
5. Delivers the final book via email if successful, else sends to telegram
group for manual inspection

## Server API Documentation

The server component provides a FastAPI backend with the following endpoints:

### Public Facing Endpoints

#### POST `/webhook`

**Purpose**: Handles Stripe payment webhooks

**Headers**:
- `stripe_signature`: Stripe signature for webhook verification

**Returns**:
- `200 OK`: `{"status": "success"}`
- `401 Unauthorized`: `{"detail": "Invalid Stripe signature"}`

**Description**: On successful payment, stores the Stripe customer_id in the database for future book generation authorization.

#### POST `/generate-book/{customer_id}`

**Purpose**: Initiates book generation process

**Path Parameters**:
- `customer_id`: Unique customer identifier from Stripe (5-30 chars), coming from the client's browser.

**Body Parameters**:
- `book_parameters`: Book customization details (relationship information)
- `uploaded_couple_image`: Image file of the couple
- `receive_book_email`: Email address of the client to receive the generated book

**Returns**:
- `200 OK`: `{"status": "success", "message": "book is in the factory!"}`
- `401 Unauthorized`: `{"detail": "customer_id not in database"}`
- `403 Forbidden`: `{"detail": "max attempts (N) exceeded"}`

**Description**: Validates customer authorization, checks attempt limits, and starts asynchronous book generation.

#### Administrative Data Management Endpoints

The system includes several administrative endpoints for data lifecycle management:

- **`/try-again/{customer_id}`**: Administrative endpoint for retrying book generation using stored customer data. Implements secure authentication and handles edge cases in the PDF generation pipeline.
- **`/modify-customer-data/{customer_id}`**: Secure interface for modifying stored customer information including email addresses and book parameters. Includes proper data validation and audit logging.
- **`/delete-all-customers-data`**: Secure bulk data deletion with administrative authentication
- **`/customer-data/{customer_id}`**: Customer data retrieval for administrative review
- **`/purge-database`**: Database maintenance operations with proper safeguards
- **`/retrieve-database`**: Secure data export functionality
- **`/delete-all-uploaded-images`**: Media asset cleanup operations

These endpoints implement:
- Multi-factor administrative authentication
- Comprehensive audit logging
- Data validation and integrity checks
- Secure communication protocols
- Automated backup verification before destructive operations

*Note: Detailed endpoint specifications are omitted for security purposes. These operations require elevated privileges and implement additional safety measures.*



### Book Generation Pipeline

1. **Payment Processing**:
   - Customer completes payment via Stripe
   - Stripe webhook notifies server
   - Server stores customer_id in database

2. **Book Generation Request**:
   - Customer submits relationship details and couple image via `/generate-book/{customer_id}`
   - Server validates customer_id and attempt limits
   - Customer data and uploaded image are saved to secure storage with proper file isolation

3. **Asynchronous Processing**:
   - Server launches background process for book generation
   - FastAPI endpoint returns immediately
   - Book generation continues in background

4. **Book Creation**:
   - Text generation creates 10 personalized chapters using GPT-4 with custom prompts
   - Image generation creates AI-generated illustrations for each chapter using DALL-E
   - HTML template engine combines text and images with responsive CSS styling
   - PDF rendering pipeline using WeasyPrint for HTML-to-PDF conversion
   - Post-processing with GhostScript for compression and optimization
   - Quality assurance checks for text coherence and image placement

5. **Delivery**:
   - On success: Book is emailed to customer with secure attachment handling
   - On failure: Error reports and partially generated content sent to monitoring system
   - Cleanup: Temporary files are securely deleted while preserving customer data for potential retries
   - Data retention: Customer information stored according to configurable retention policies

### Error Handling

The system implements comprehensive error tracking throughout the generation process:
- **Text Generation Failures**: Implements fallback mechanisms with alternative prompts and retry logic
- **Image Generation Failures**: Graceful degradation with placeholder images and error documentation
- **PDF Processing Failures**: Multi-stage rendering pipeline with compression fallbacks
- **Critical System Errors**: Real-time monitoring with automated alerting to administrative channels
- **Data Integrity Checks**: Validation at each pipeline stage with rollback capabilities
- **Performance Monitoring**: Request timing, resource usage, and bottleneck identification

### Technical Architecture

**Backend Framework**: FastAPI with asynchronous request handling and automatic API documentation

**AI Integration**: 
- OpenAI for both images and text generation.
- Highly Custom prompt engineering for relationship-specific, consistent content generation

**Document Processing Pipeline**:
- Custom HTML template placeholding with customer information
- WeasyPrint for HTML-to-PDF conversion with custom CSS
- GhostScript integration for PDF optimization and compression

**Data Management**:
- Secure file storage with proper access controls and encryption
- Database integration for customer tracking and analytics
- Automated backup and recovery systems
- GDPR-compliant data handling with configurable retention policies
