## Forem generator error

The error is reported here: https://github.com/radar/forem/issues/495

### The Error
`undefined method `forem_moderate_posts?' for #<User:0x007ff1637c0f40> (NoMethodError)`


### To Reproduce

1. Run `bundle install`.
2. Run `rake db:migrate` to create the User table.
3. Run `rake db:seed` to create a User.
4. Run `rails g forem:install` and use the default options.
5. You should see the error.


### Why?

##### Conditions
1. `Forem.user_class` isn't defined when you run the installer. This could either be because you're running it for the first time, or because you deleted the `forem.rb` initializer. Either way, [this condition](https://github.com/radar/forem/blob/1daa9135b56f4bd6ec05dcc4c011403b5da2d9d6/app/decorators/lib/forem/user_class_decorator.rb#L2) cannot pass, and therefore doesn't add the decorate methods to the `User` class (such as `forem_moderate_posts?`).
2. At least one `User` record already exists in the database. When this is true, [this condition](https://github.com/radar/forem/blob/1daa9135b56f4bd6ec05dcc4c011403b5da2d9d6/db/seeds.rb#L4) passes, and a `Post` is created, [which calls the `forem_moderate_posts?` method](https://github.com/radar/forem/blob/1daa9135b56f4bd6ec05dcc4c011403b5da2d9d6/app/models/forem/post.rb#L116) in a callback.

##### Cause
By the time the generator gets to the [`seed_database`](https://github.com/radar/forem/blob/1daa9135b56f4bd6ec05dcc4c011403b5da2d9d6/lib/generators/forem/install_generator.rb#L64) task, the environment has already been loaded (including the `to_prepare` and `after_initialize` callbacks, the two places where [decorators are loaded](https://github.com/parndt/decorators/blob/18defc9a2c1b5749bf1ec0ff31465d943932ef7b/lib/decorators/railtie.rb#L6)), but when the `decorators` gem tried to load the `user_class_decorator` file earlier, it didn't actually do anything because `Forem.user_class` hadn't been set yet.


### Solution
If you're experiencing this problem and it hasn't been fixed in the `forem` gem yet, then just manually create `config/initializers/forem.rb` and copy the contents of [this file](https://github.com/radar/forem/blob/1daa9135b56f4bd6ec05dcc4c011403b5da2d9d6/lib/generators/forem/install/templates/initializer.rb) into it. Then change the `<%= ... %>` bits to something appropriate, like "User".
