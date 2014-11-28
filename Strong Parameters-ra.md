
#Strong Parameters

##Introduction

**Mass Assignment in Rails 2**
- Vulnerability

**Blacklisting and Whitelisting in Rails 3**

- attr_protected =>  Blacklisting
- attr_accessible =>  whitelisting
- Problem :
 
> One difficulty with whitelisting was that for the model to have different levels of accessibility, you would need an attr_accessible declaration for each level.

**Strong Parameters in Rails 4**
- Strong Parameters force us to take logic out of the model.
- It still uses whitelisting, but this has been moved out of the model.
- Before passing parameters into the model, they need to be wrapped in a call to permit.
- Permit different parameters for different roles
- The less difficult it is both to test and to modify
- Delegate the behavior controlling access into its own object
adds an extra layer of security that prevents attackers from posting harmful or garbage information to your site.


##History
- Posted by dhh, March 21, 2012
- Add to Rail Core in 2014 (Rail 4.0)

##Installation
 ```ruby
    gem 'strong_parameters'
```

To activate the strong parameters, you need to include this module in every model you want protected.
 
```ruby
class Post < ActiveRecord::Base
  include ActiveModel::ForbiddenAttributesProtection
end
```
Alternatively, you can protect all Active Record resources by default by creating an initializer and pasting the line:
 ```ruby
ActiveRecord::Base.send(:include, ActiveModel::ForbiddenAttributesProtection)
 ```
If you want to now disable the default whitelisting that occurs in Rails 3.2, change the config.active_record.whitelist_attributes property in your config/application.rb:
 ```ruby
config.active_record.whitelist_attributes = false
 ```
This will allow you to remove / not have to use attr_accessible and do mass assignment inside your code and tests.

## Example

 ```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributes exception
  # because it's using mass assignment without an explicit permit
  # step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key
  # in the parameters, otherwise it'll raise a
  # ActionController::ParameterMissing exception, which will get
  # caught by ActionController::Base and turned into that 400 Bad
  # Request reply.
  def update
    person = current_account.people.find(params[:id])
    person.update!(person_params)
    redirect_to person
  end

  private
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
 ```

### Nested Parameters
 ```ruby
params.permit(:name, { emails: [] },
              friends: [ :name,
                         { family: [ :name ], hobbies: [] }])
 ```

**accepts_nested_attributes_for** allows you to update and destroy associated records. This is based on the id and _destroy parameters:
 ```ruby
# permit :id and :_destroy
params.require(:author).permit(:name, books_attributes: [:title, :id, :_destroy])
 ```
Hashes with integer keys are treated differently and you can declare the attributes as if they were direct children. You get these kinds of parameters when you use accepts_nested_attributes_for in combination with a has_many association:

 ```ruby
# To whitelist the following data:
# {"book" => {"title" => "Some Book",
#             "chapters_attributes" => { "1" => {"title" => "First Chapter"},
#                                        "2" => {"title" => "Second Chapter"}}}}

params.require(:book).permit(:title, chapters_attributes: [:title])
 ```

###Compatibility

- An unofficial Rails 2 version is strong_parameters_rails2.
- Rails versions 3.0, 3.1 and 3.2
- It is part of Rails Core in 4.0.
