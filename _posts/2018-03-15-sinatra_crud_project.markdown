---
layout: post
title:      "Sinatra CRUD Project"
date:       2018-03-15 18:48:14 +0000
permalink:  sinatra_crud_project
---


Creating a Create-Read-Update-Delete (CRUD) application with Sinatra is mostly applying Ruby/HTML/CSS skills. What is new and interesting is collecting user input in a useful way and establishing data persistence & associations through ActiveRecord.

"The Great British Baking Show" was recently recommended to me, and I've since been keeping an eye out for opportunities to try what I've seen on the show. Baking and programming are similar in many ways: the smallest mistake along the way can have drastic and unforeseen effects on the final product, and everyone has their own means toward the same end. My CRUD app will give users the ability to read everyone's recipes and create, read, update, and delete their own.

When dealing with associations it is critical to plan ahead. I sketched out my app as follows before even touching the keyboard:

First I identified the required objects and the attributes that they possess *in isolation*:
* User: email, password
* Recipe: name, instructions, user
* Ingredient: name

Now obviously an ingredient needs a quantity, but when you really think about it the quantity does not belong to the ingredient. Everything worthwhile in baking will have sugar, but most recipes will require different quantities of sugar. We have uncovered a "many-to-many" relationship; ubiquitous in real life but a little tricky to translate into code. The quantity attribute actually belongs to an ingredient in the context of a specific recipe, a realization that leads to another object we'll need: RecipeIngredient.

Identified Object List:
* User: email, password
* Recipe: name, instructions, user
* Ingredient: name
* RecipeIngredient: recipe, ingredient, quantity

Next I map out how each object needs to associate with each other object:

User
* Recipes: has many
* Ingredient: don't need to track an association
* RecipeIngredient: don't need to track an association

Recipe
* User: belongs to
* RecipeIngredient: has many
* Ingredient: has many through RecipeIngredients

Ingredient
* User: don't need to track an association
* RecipeIngredient: has many
* Recipe: Has many through RecipeIngredients

RecipeIngredient
* User: don't need to track an association
* Recipe: belongs to
* Ingredient: belongs to

Having planned ahead I can start programming and implementing these relationships with a clear head.

Once the object architecture is established, the next challenge is collecting user input. I designed forms that produce the following params hash:
```
params = {
   recipe: {name: "user input", instructions: ["user input"] }, 
	 ingredients: [{name: "user input", quantity: "user input"}]
	 } 
```

I can then create a recipe (storing the instructions array as a | separated string) with:
```
recipe = Recipe.create(
   name: params[:recipe][:name], 
   instructions: params[:recipe][:instructions].join("|")
	 )
```

And find or create all of the recipe's ingredients with:
```
ingredients = params[:ingredients].collect {|ingredient_hash|
   Ingredient.find_or_create_by(name: ingredient_hash[:name])
	 }
```

Collect all the ingredient quantities with:
```
quantities = params[:ingredients].collect {|ingredient_hash| ingredient_hash[:quantity]}
```

Establish the recipe-ingredient many-to-many relationship with:
```
i=0
while i < ingredients.length do
     RecipeIngredient.create(recipe: recipe, ingredient: ingredients[i], quantity: quantities[i])
     i += 1
end
```

And finally associate the recipe with it's user with:
```
user = User.find(session[:user_id])
user.recipes << recipe
user.save
```

Done this way the user only has to fill out one form and all of our objects are created with the proper associations.





