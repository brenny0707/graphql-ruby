# GraphQL Notes

## Create User Steps with Auth for Email

1. We created a `Type` called `AuthProviderEmailInput` with string args of `email` and `password`.

```ruby
Types::AuthProviderEmailInput = GraphQL::InputObjectType.define do
  name 'AUTH_PROVIDER_EMAIL'

  argument :email, !types.String
  argument :password, !types.String
end
```

### Steps 2 & 3

```ruby
class Resolvers::CreateUser < GraphQL::Function
  AuthProviderInput = GraphQL::InputObjectType.define do
    name 'AuthProviderSignupData'

    argument :email, Types::AuthProviderEmailInput
  end

  argument :name, !types.String
  argument :authProvider, !AuthProviderInput

  type Types::UserType

  def call(_obj, args, _ctx)
    User.create!(
      name: args[:name],
      email: args[:authProvider][:email][:email],
      password: args[:authProvider][:email][:password]
    )
  end
end
```

### Breaking this resolver down

2a. We then create a resolver that takes a `name` string arg, and instead of passing `email` and `password` args as a “top-level” params, we call an `authProvider` arg:

```ruby
  argument :name, !types.String
  argument :authProvider, !AuthProviderInput
```

that we define in the lines above as `AuthProviderInput` with:
```ruby
AuthProviderInput = GraphQL::InputObjectType.define do
    name 'AuthProviderSignupData'

    argument :email, Types::AuthProviderEmailInput
  end
```


2b. `AuthProviderInput` takes an `email` arg that is an `AuthProviderEmailInput` type (the type we created in step 1), which is where we pass the `email` and `password` args.

3\. In the `call` method, the args for `name` is just `[:name]`, while `email` and `password`  go through `args[:authProvider][:email]`, since the `AuthProviderInput`’s argument is `email`, and the third args `email` and `password` are the actual email and password strings passed to the `AuthProviderEmailInput` type.