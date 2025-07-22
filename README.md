# JSO Specification - Draft 1.0
**Last Updated: July 22, 2025**

A simple, pragmatic, and scalable JSON response format specification.

## Introduction

The JSO Specification was born out of a need for a common-sense standard for API responses that sits comfortably between overly simplistic formats and highly complex ones. The name 'JSO' is a nod to its roots: **J**SON from **S**tack **O**verflow.

### The Motivation

1. **Inspiration:** This project was heavily inspired by a [Stack Overflow question](https://stackoverflow.com/questions/12806386/is-there-any-standard-for-json-api-response-format) and the thoughtful answers from the community. It became clear that many developers are seeking a simple, intuitive "envelope" format for their JSON responses.

2. **The Middle Ground:** While specifications like [JSend](https://github.com/omniti-labs/jsend) are excellent, they can have minor ergonomic issues. On the other end of the spectrum, specifications like [JSON:API](https://jsonapi.org/) are incredibly powerful but often too verbose and complex for rapid prototyping or simpler projects. JSO aims to be the perfect middle ground.

3. **Developer Experience:** Many in the community would likely agree that JSend is *almost* perfect for simple and starter projects. However, a common annoyance in JavaScript is the need to check for success with a string comparison (`if (jsonResp.status === "success")`) rather than a more natural boolean check (`if (jsonResp.success)`). JSO adopts the boolean approach for simplicity and better developer ergonomics. A relatively loud shout-out to the JS Developers out there!

### The Inevitable Standards Dilemma

I'm fully aware of the irony of creating a new standard.

![XKCD: Standards](https://imgs.xkcd.com/comics/standards.png)

The goal of JSO is not to replace other standards but to provide a well-defined and documented alternative for those who share the same design philosophy.

## Specification

### General Structure

A JSO response is a JSON object that **MUST** contain a top-level boolean `success` property.

#### 1. Successful Responses (`success: true`)

When a request is successful, the `success` property **MUST** be `true`.

A successful response **SHOULD** contain a top-level `data` key.

**Example:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Doe"
  }
}
```

#### 2. Failed Responses (`success: false`)

When a request fails, the `success` property **MUST** be `false`.

A failed response **MUST** contain a top-level `message` key with a string value describing the error. This is intended for simple, direct feedback.

**Simplest Error Response:**
```json
{
  "success": false,
  "message": "The requested resource could not be found."
}
```

For more detailed errors, a failed response **MAY** include an optional `errors` key, which contains an array of error objects. This is useful for scenarios like form validation where multiple issues can occur at once.

**Detailed Error Response:**
```json
{
  "success": false,
  "message": "Invalid input provided. Please check the details.",
  "errors": [
    {
      "code": 101,
      "message": "Email is invalid.",
      "source": { "field": "email" }
    },
    {
      "code": 202,
      "message": "Password must be at least 8 characters.",
      "source": { "field": "password" }
    }
  ]
}
```

### Key Definitions and Rules

#### 1. The `data` Field

* **In Successful Responses:** The `data` field contains the primary resource(s) from the request.
    * For a single resource (`GET /users/1`), `data` **MUST** be an object.
    * For a collection of resources (`GET /users`), `data` **MUST** be an array of objects. If the collection is empty, it **MUST** be an empty array (`[]`).
* **In Failed Responses (Optional but Recommended):** In a `success: false` response, the `data` field, if present, is **highly recommended** to contain the original data sent by the client that caused the error. This is optional to allow for faster prototyping but provides valuable context for clients to re-populate forms.

#### 2. Handling "Not Found" Resource(s)

* **Collections:** A request for a collection that has no items is considered a success. It **MUST** return `success: true` with an empty array for the `data` field.
* **Single Resources:** A request for a single resource that does not exist is considered a failure. It **MUST** return a `success: false` response with an appropriate `message` (e.g., "Resource not found.").

#### 3. Distinguishing Error Types (Optional)

To distinguish between client-side errors (e.g., bad input) and server-side errors (e.g., database failure), an error object within the optional `errors` array **MAY** include a `type` field.

**Example of a Client-Side Error:**
```json
{
  "success": false,
  "message": "Invalid input provided.",
  "errors": [
    {
      "type": "VALIDATION_ERROR",
      "code": 101,
      "message": "Email is invalid."
    }
  ]
}
```

**Example of a Server-Side Error:**
```json
{
  "success": false,
  "message": "An internal server error occurred.",
  "errors": [
    {
      "type": "SERVER_ERROR",
      "code": 5001,
      "message": "Failed to connect to the database."
    }
  ]
}
```

#### 4. Metadata and Pagination (Optional but Recommended)

For complex responses, particularly those involving collections that require pagination, it is **highly recommended** to include optional top-level `meta` and `links` objects.

* `meta`: An object containing non-standard meta-information about the response (e.g., pagination details, request IDs).
* `links`: An object containing links to related resources or pagination controls.

**Example with Pagination:**
```json
{
  "success": true,
  "data": [
    { "id": 6, "title": "Post Title 6" },
    { "id": 7, "title": "Post Title 7" },
    { "id": 8, "title": "Post Title 8" },
    { "id": 9, "title": "Post Title 9" },
    { "id": 10, "title": "Post Title 10" }
  ],
  "meta": {
    "totalItems": 150,
    "itemsPerPage": 5,
    "currentPage": 2
  },
  "links": {
    "self": "/api/posts?page=2",
    "first": "/api/posts?page=1",
    "prev": "/api/posts?page=1",
    "next": "/api/posts?page=3",
    "last": "/api/posts?page=30"
  }
}
```

## JavaScript Usage

To simplify working with JSO-compliant APIs, an official helper library, `jso-client`, is available. It is a lightweight, zero-dependency wrapper around the Fetch API.

The client can be found on GitHub: [https://github.com/mannyvergel/jso-client](https://github.com/mannyvergel/jso-client)

### Installation
```bash
npm install jso-client
```

### Example
The client automatically handles the JSO envelope, returning the `data` on success, and throwing a structured `JsoError` on failure.

```javascript
import { jsoFetch, JsoError } from 'jso-client';
// Or: const { jsoFetch, JsoError } = require('jso-client');

async function getUser(id) {
  try {
    const { data: user } = await jsoFetch(`/api/user/${id}`, { method: 'POST' });
    console.log('User:', user);
  } catch (error) {
    console.error('Request Failed:', error.message);
  }
}
```

For more advanced options and error handling, please check the docs of jso-client.

## Community and Contribution

I welcome the community to contribute and improve on this initial specification. Please feel free to open issues or pull requests on the GitHub repository.

## License

This specification is licensed under the **MIT License**.

## Credits

This specification was made possible by the inspiration and ideas shared by the following Stack Overflow members: FtDRbwLXw6, prusswan, Adam Gent, robert_difalco, and Syed.
