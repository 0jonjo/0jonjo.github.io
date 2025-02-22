---
layout: post
title: "Implementing JSON Web Token in Ruby on Rails"
description: "How to code and decode JWT tokens in a Rest API"
tags: ruby english
categories: misc
image: https://nordicapis.com/wp-content/uploads/Why-Cant-I-Just-Send-JWTs-Without-OAuth-JWT.png
---

An obligatory theme in the world of back-end web development is user authentication. In the context of a Ruby on Rails project, the most common way to authenticate users is through the [use of sessions](https://guides.rubyonrails.org/v4.1/action_controller_overview.html#session). However, there are other ways to authenticate, such as JSON Web Token (JWT). I'm currently working on a project that uses JWT for authentication in some web frontend and Android apps. I've been studying this to understand some parts of the legacy code, reading articles, docs, and practicing in one of my personal projects, [Alljobs](https://github.com/0jonjo/alljobs). There are many articles and tutorials about JWT, and I'll share the most useful ones that I've found and summarize the main points of my implementation in this post.

### Let's start

To begin, it's useful to know the difference between authorization and authentication. Authentication is the process of verifying who a user is. On the other hand, authorization is the process of verifying what they have access to. JWT is a way to authenticate users, not to authorize them. It's a standard for creating and consuming tokens in a secure way. It's a compact, URL-safe means of representing claims (data) used to authenticate users on a website or app.

JWT works like a ping-pong game between the client and the server. The client sends a request to the server with the user's credentials. The server validates the credentials, generates a token, and sends it to the client. The client stores the token and sends it in the header of each request. The server validates the token and sends the response. Diagram of the JWT flow request-response from [Bluebash](https://bluebash.co/):

<table cellpadding="0" cellspacing="0" border="0" width="100%">
<tr><td align="center">
  <img src="https://www.bluebash.co/blog/content/images/2021/12/image-5.png" width="450">
</td></tr>
</table>

What's the ball of the game? The token. It's a string that contains three parts: header, payload, and signature. The token is composed of three parts separated by dots. The first part is the header, the second part is the payload, and the third part is the signature. The header and payload are JSON objects that are base64 encoded. The signature is the encoded header, encoded payload, a secret, the algorithm specified in the header, and the signing method. Example of a JWT token:

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Image with JWT token parts from [Nordic APIs](https://nordicapis.com/):

<table cellpadding="0" cellspacing="0" border="0" width="100%">
<tr><td align="center">
  <img src="https://nordicapis.com/wp-content/uploads/Why-Cant-I-Just-Send-JWTs-Without-OAuth-JWT.png" width="450">
</td></tr>
</table>

### Show me the code

I've implemented JWT in [Alljobs](https://github.com/0jonjo/alljobs). It's a job opening website built using Ruby on Rails. The project is open-source, and you can check all the code in the repository. I'll show the main parts of the implementation here.

First, I've added the [`jwt gem`](https://github.com/jwt/ruby-jwt) to the Gemfile and installed it. It's a Ruby implementation of the JSON Web Token (JWT) standard. It's a simple library to encode and decode JWT tokens with useful instructions on how to use and examples in the Readme.

Now, with the power to handle JWT tokens, I've created the [`JsonWebToken`](https://github.com/0jonjo/alljobs/blob/main/lib/json_web_token.rb) class in lib for encoding and decoding the JWT tokens. The code is shown below, pay attention to the encode method that adds an expiration time - "exp" - to the token and the use of the Rails secret key base to encode and decode the token. In this implementation, you do not hard-code the key or store it in an environment variable; you put it in a more secure place, the Rails credentials. This is a minimal explanation of [how and why to do that](https://medium.com/@V-Blaze/rails-7-encrypted-credentials-elevating-security-in-your-application-5a84a669bf18). The key guarantees the security of the token; it's used to encode and decode it, so it's important to keep it safe.

```ruby
class JsonWebToken
  class << self
    def encode(payload, exp = 24.hours.from_now)
      payload[:exp] = exp.to_i
      JWT.encode(payload, Rails.application.credentials.secret_key_base)
    end

    def decode(token)
      JWT.decode(token, Rails.application.credentials.secret_key_base)
    rescue StandardError
      nil
    end
  end
end
```

In the decode method, I've used a rescue block to return nil if the token is invalid. We do that to handle the error in another part of the code. The token can be invalid for many reasons, such as the token is expired, the token is tampered with, the token is not signed with the correct key, etc. Now that we have the encode method, we can use it to generate the token for the user. I've created an [`Authenticate`](https://github.com/0jonjo/alljobs/blob/main/app/models/concerns/authenticate.rb) class to handle the authentication.
It has an initialize method that receives the email and password of the user. The user method checks if the user exists and if the password is correct. If the user exists and the password is correct, the authenticate method is called and generates the call_user; if not, it raises an error. In the Alljobs implementation, there are User and also Headhunter, but the logic is the same. The code is shown below:

```ruby
class Authenticate
  def initialize(email, password, account_type)
    @email = email
    @password = password
    @account_type = account_type
  end

  def call
    authenticate!

    JsonWebToken.encode(@payload)
  end

  private

  attr_reader :email, :password

  def authenticate!
    @account = @account_type.constantize.find_by(email: email)
    raise ActiveRecord::RecordNotFound unless @account&.valid_password?(password)

    @account_type == "User" ? user_payload : headhunter_payload
  end

  def user_payload
    @payload = { user_id: @account.id }
  end

  # Other method to headhunter_payload
end
```

The "authenticate!" method is responsible for checking if the user exists and if the password is correct. If the user exists and the password is correct, the payload is generated. The payload is a hash that contains the user or headhunter id; it's used to generate the token. Note that the payload does not contain the password; it's a secure way to ensure that the system does not have a method to access the password directly. The `Authenticate` class is used in the [`TokensController`](https://github.com/0jonjo/alljobs/blob/main/app/controllers/api/v1/tokens_controller.rb), the controller that is responsible for generating the token for the user. Depending on the user type, the controller calls the `Authenticate` class with the correct user type. The code is shown below:

```ruby
class TokensController < Api::V1::ApiController
  def auth_user
    authenticate("User")
  end

  def auth_headhunter
    authenticate("Headhunter")
  end

  private

  def authenticate(user_type)
    auth = Authenticate.new(params[:email], params[:password], user_type).call
    auth_user_type(user_type, auth)
  rescue ActiveRecord::RecordNotFound
    log_and_return_error(user_type)
  end

  # Other methods to log and return errors
end
```

When the user sends the email and password to the server, the controller calls the `Authenticate` class with the correct user type. If the user exists and the password is correct, the token is generated and returned to the user; if not, an error is returned. The user must store the token and send it in the header of each request. The server must validate the token in each request. I've created an [`Authenticable`](https://github.com/0jonjo/alljobs/blob/main/app/controllers/concerns/authenticable.rb) module with methods to validate the token and check if it is not expired (remember that we put a 24-hour period). The code is shown below:

```ruby
module Authenticable
  def authenticate_with_token
    @token ||= request.headers["Authorization"]

    render_unauthorized unless valid_token?
  end

  def valid_token?
    body = JsonWebToken.decode(@token)
    body[0]["exp"] > Time.now.to_i if body
  end

  # Other method to render unauthorized
end
```

The `authenticate_with_token` method is called in the controller that needs to validate the token and check if it is expired. The `Authenticable` module is included in the `ApiController` to be used in all controllers that need to validate the token.

### Conclusion

We've seen that JWT is a powerful tool to authenticate users in a secure way. We learned how to encode and decode the token, how to handle the authentication, how to generate the token, and how to validate the token. You can check the full implementation in the [Alljobs repository](https://github.com/0jonjo/alljobs). I hope this post helps you understand how to use JWT in a Ruby on Rails project. If you have any questions or suggestions, please let me know.

### References

- [JWT Ruby Gem](https://github.com/jwt/ruby-jwt)
- [JWT documentation](https://datatracker.ietf.org/doc/rfc7519/)
- [Token-based Authentication with Ruby on Rails](https://www.pluralsight.com/resources/blog/guides/token-based-authentication-with-ruby-on-rails-5-api)
- [Rails 6 and 7 API Authentication](https://medium.com/@ronakabhattrz/rails-6-7-api-authentication-with-jwt-token-based-authentication-2e2d22a70643)
- [A Complete Guide to Rails Authentication Using JWT](https://dev.to/mohhossain/a-complete-guide-to-rails-authentication-using-jwt-403p)
- [The Complete Guide to Rails Credentials](https://webcrunch.com/posts/the-complete-guide-to-ruby-on-rails-encrypted-credentials)
