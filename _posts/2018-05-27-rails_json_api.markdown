---
layout: post
title:      "Rails JSON API"
date:       2018-05-27 18:10:34 +0000
permalink:  rails_json_api
---


Robust front-end frameworks like React has created the need to strip Rails down to only what is needed to serve as an API--basically models and controllers without the views. It turns out that much of Rails is still useful even without the view layer. This post will walk through creating a JSON API for associated models.

Start by creating a rails api app and models as usual:
```
Console:
rails new app_name --api
rails g model model_a attr_1:dataType attr_n:dataType
rails g model model_b attr_1:dataType attr_n:dataType
rails g model model_a_model_b model_a_id:integer model_b_id:integer attr_n:dataType
```

Set up associations:
```
app_name/app/models/model_a.rb:
  class ModelA < ApplicationRecord
	  has_many :model_a_model_bs
		has_many :model_bs, through: :model_a_model_bs
	end
		
app_name/app/models/model_b.rb:
  class ModelB < ApplicationRecord
	  has_many :model_a_model_bs
		has_many :model_as, through: :model_a_model_bs
  end
	
app_name/app/models/model_a_model_b.rb:
  class ModelB < ApplicationRecord
	  belongs_to :model_a
		belongs_to :model_b
	end
```

ActiveModel Serializers relies on Rails conventions to make serializing models into JSON a breeze.
```
app_name/app/Gemfile:
gem 'active_model_serializers'

Console:
bundle install
rails g serialzer model_a

app_name/app/serializers/model_a.rb
  class ModelASerializer < ActiveModel::Seralizer
	  attributes :attr_1, :attr_n
```

This will create JSON for any model_a instance as follow
```
{
  "attr_1": attr_1_value,
	"attr_n": attr_n_value
}
```

Associated models can easily be included
```
app_name/app/serializers/model_a.rb
  class ModelASerializer < ActiveModel::Seralizer
	  attributes :attr_1, :attr_n
		has_many :model_bs
```

This will create JSON for any model_a instance as follows
```
{
  "attr_1": attr_1_value,
	"attr_n": attr_n_value,
	"model_bs": [{
	  "attr_1": attr_1_value,
		"attr_n": attr_n_value
	}]
}
```

Seralized attributes are also easily customizable
```
app_name/app/serializers/model_a.rb
  class ModelASerializer < ActiveModel::Seralizer
	  attributes :attr_1, :attr_n
		has_many :model_bs
		attribute :custom_attribute do
		  // code to generate custom attribute, be careful to return valid JSON
		end
```

Lastly, the JSON must be exposed so it can be accessed by external applications (in the following case at app_url/modela/:id)
```
app_name/config/routes.rb
  Rails.application.routes.draw do
	  get '/modela/:id', to: model_as#controller_action, as: nil
	end

app_name/app/controllers/model_as_controller.rb
  class ModelAsController < ApplicationController
	  def controller_action
		  @modelA = ModelA.find(params[:id])
			render json: @modelA
		end
	end
```

