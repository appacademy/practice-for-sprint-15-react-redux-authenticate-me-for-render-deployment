# Authenticate Me - Backend (Pre-frontend)

Before you start your frontend, there's a few things you need to make sure
you have in your backend.

## Get Current User Endpoint

Your get current user endpoint, `GET /api/session`, should return 2 different
responses depending on if there is a current user logged into your application
or not.

If there is **NO** current user logged in, the endpoint should return the
following JSON body **in this exact format**:

```json
{
  "user": null
}
```

Make sure there is a key of `user` that has a value of `null`.

If there is a current user logged in, the endpoint should return the
following JSON body **in this exact format**:

```json
{
  "user": {
    "id": 1,
    "firstName": "John",
    "lastName": "Smith",
    "email": "john.smith@gmail.com",
    "username": "JohnSmith",
  }
}
```

**NOTE**: The `user.token` key is optional and not required.

## Log In a User endpoint

Your log in endpoint, `POST /api/session`, should return the following JSON
body **in this exact format** if the log in attempt is successful:

```json
{
  "user": {
    "id": 1,
    "firstName": "John",
    "lastName": "Smith",
    "email": "john.smith@gmail.com",
    "username": "JohnSmith"
  }
}
```

The `user.token` key is optional and not required.

## Sign Up a User endpoint

Your sign up endpoint, `POST /api/users`, should return the following JSON
body **in this exact format** if the sign up attempt is successful:

```json
{
  "user": {
    "id": 1,
    "firstName": "John",
    "lastName": "Smith",
    "email": "john.smith@gmail.com",
    "username": "JohnSmith"
  }
}
```

**NOTE**: The `user.token` key is optional and not required.

## Final Confirmation

Make sure to test the endpoints above to make sure they work as expected on
Postman or through the browser. **These requirements are necessary to go through
the next frontend instructions smoothly.**