# API Documentation Journal: User Registration Endpoint

**Student Name:** Seshoka Mailula  
**Student Email:** smailula40@ai.wethinkcode.co.za  
**Date:** May 2026  

---

## 🛠️ Section 1: Original API Endpoint Code

```python
@app.route('/api/users/register', methods=['POST'])
def register_user():
    """Register a new user"""
    data = request.get_json()

    # Validate required fields
    required_fields = ['username', 'email', 'password']
    for field in required_fields:
        if field not in data:
            return jsonify({
                'error': 'Missing required field',
                'message': f'{field} is required'
            }), 400

    # Check if username or email already exists
    if User.query.filter_by(username=data['username']).first():
        return jsonify({
            'error': 'Username taken',
            'message': 'Username is already in use'
        }), 409

    if User.query.filter_by(email=data['email']).first():
        return jsonify({
            'error': 'Email exists',
            'message': 'An account with this email already exists'
        }), 409

    # Validate email format and password strength
    if not re.match(r"^[^@]+@[^@]+\.[^@]+$", data['email']):
        return jsonify({'error': 'Invalid email', 'message': 'Please provide a valid email address'}), 400
    if len(data['password']) < 8:
        return jsonify({'error': 'Weak password', 'message': 'Password must be at least 8 characters long'}), 400

    # Create new user logic...
    try:
        password_hash = generate_password_hash(data['password'])
        new_user = User(username=data['username'], email=data['email'].lower(), password_hash=password_hash, created_at=datetime.utcnow(), role='user')
        db.session.add(new_user)
        db.session.commit()
        return jsonify({'message': 'User registered successfully', 'user': {'username': new_user.username}}), 201
    except Exception as e:
        db.session.rollback()
        return jsonify({'error': 'Server error', 'message': 'Failed to register user'}), 500

        ## Section 2: Comprehensive Endpoint Documentation

        ## 📝 Endpoint Documentation: User Registration

**Endpoint:** `POST /api/users/register`

**Purpose:** Registers a new user account by validating input, checking for existing credentials, and securing user data via hashing.

### Request Parameters
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `username` | string | Yes | Unique identifier for the user |
| `email` | string | Yes | Valid email address |
| `password` | string | Yes | Password (must be >= 8 characters) |

### Responses
* **201 Created:** User successfully registered.
* **400 Bad Request:** Validation failure (missing fields, invalid email, weak password).
* **409 Conflict:** Username or email already registered.
* **500 Internal Server Error:** Database or system error.

### Examples
**Request:**
```http
POST /api/users/register
Content-Type: application/json

{
  "username": "test",
  "email": "test@test.com",
  "password": "password123"
}

## Section 3: OpenAPI/Swagger Reference
openapi: 3.0.0
info:
  title: User Registration API
  version: 1.0.0
paths:
  /api/users/register:
    post:
      summary: Register a new user
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                username: {type: string}
                email: {type: string}
                password: {type: string}
      responses:
        '201': {description: "User registered"}
        '400': {description: "Validation error"}
        '409': {description: "Conflict"}
        '500': {description: "Server error"}

## 📖 Section 4: Developer Usage Guide

1.  **Authentication:** No authentication is required for this public registration endpoint.
2.  **Request Format:** Send a `POST` request with a `Content-Type: application/json` header and a valid JSON body.
3.  **Troubleshooting:** * **400 Errors:** Validate that your JSON fields match the required schema (username, email, password).
    * **409 Errors:** Ensure the username and email are unique; try a different username or check if you already have an account.

---

## 🚀 Section 5: Exercise Reflection

* **Most challenging part:** Distinguishing between API business rules (like password strength) and system-level errors (server errors).
* **Prompt Engineering Strategy:** I had to explicitly ask for the error code scenarios (400, 409, 500) to ensure the documentation was comprehensive and useful for debugging.
* **Format Effectiveness:** OpenAPI is significantly more effective than standard Markdown for large teams, as it allows tools to automatically generate client-side code, whereas Markdown is better for human-readable "Getting Started" guides.
* **Workflow Integration:** I would automate this documentation generation as part of the CI/CD pipeline so that documentation updates whenever the API endpoint code changes.        
