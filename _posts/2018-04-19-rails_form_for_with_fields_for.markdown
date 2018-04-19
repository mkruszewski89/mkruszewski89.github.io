---
layout: post
title:      "Rails form_for with fields_for"
date:       2018-04-19 17:36:47 +0000
permalink:  rails_form_for_with_fields_for
---


Users must be able to input data intuitively and simply; developers must parse those data in ways that are not so simple. Forms bridge the simple user interface and the complex back-end by structuring inputs into consistent data structures. Rails facilitates with some magic that is revealed in this post.

form_for is a helper method that yields a form builder object for a specified model:

```
<%= form_for your_model, url: appropriate_route, method: :http_request_type do |form_builder| %>
```

The form_builder object accepts various methods for the model's attributes, for example:

```
<%= form_builder.text_field(:your_model_attribute) %>
```

form_for has a slew of other helper methods for generating html such as email_field, password_field, date_field, select, etc. When the form is submitted the controller action specified in your route is passed a nested hash in params:

```
params[:your_model][:attributes]
```

Additional model objects can be added to the same form with fields_for:

```
fields_for :your_other_model do |field_builder|
```

fields_for also yields a form builder object that can be operated on with various methods for the model's attributes. To get the additional object's attributes in params, the object specified in form_for must accept nested attributes for the object specified in fields_for:

```
class YourModel
  accepts_nested_attributes_for :your_other_model
```

On submission, params will have another nested hash (and if it is a has_many relationship, an index will be included):

```
params[:your_model][:your_other_model_attributes][index][:attributes]
```

With organized data passed to the controller you can use it in any way you see fit. This could all be done with vanilla html, and a good place to start in debugging/exploring is to inspect the resultant html, but by following rails conventions you can vastly cut down on your code base.
