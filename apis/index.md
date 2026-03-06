# API Reference

Welcome to the EOS Platform API documentation. This reference provides comprehensive details on the available services, endpoints, and data models to help you integrate seamlessly with our platform.

## Introduction

The EOS API is organized around **REST**. Our API has predictable resource-oriented URLs, accepts **JSON-encoded** request bodies, returns **JSON-encoded** responses, and uses standard HTTP response codes, authentication, and verbs.

We provide separate environments for development, testing, staging, and production. The base URL you interacts with determines which environment you are accessing.

## Authentication

Authentication to the API is performed via **Bearer Tokens** (JWT). You must include your access token in the `Authorization` header for all protected endpoints.

```http
Authorization: Bearer <your_access_token>
```

## Errors

We use standard HTTP response codes to indicate the success or failure of an API request.
- `2xx` - Success.
- `4xx` - Client error (e.g., missing parameters, invalid data).
- `5xx` - Server error.

## Available APIs

The platform is modular, and APIs are grouped by domain:

### Accounts Management
Manage user identities, organizations, and memberships. This service is backed by Okta and provides the foundation for user access and role management.

### Customer Communication
A centralized Unified Notification Service for sending transactional, promotional, and marketing messages via Email, Push, and SMS. includes tracking and webhooks.

### Flights
Access rich travel content including:
- **Flight Sales**: Search for flights, get pricing, and view amenities and baggage rules.
- **Locations**: Resolve coordinates and get autocomplete suggestions for airports and cities.
