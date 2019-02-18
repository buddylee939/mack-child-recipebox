# Steps

- rails new recipebox
- installing haml gem
- install bootstrap sass
- install jquery rails
- install simple form
- bundle
- change app.scss
- rails generate simple_form:install --bootstrap
- rails g scaffold Recipe title description:text user_id:integer {he did the controller/model way}
- rails db:migrate
- routes: root 'recipes#index'
- delete scaffold scss
- he added paperclip
- rails active_storage:install
- rails db:migrate
- add to recipe.rb model

```
has_one_attached :image
```

- update recipe form partial

```
<%= f.input :image %>
```

- update the recipes controller recipes params to include the image

```
def recipe_params
  params.require(:recipe).permit(:title, :description, :user_id, :image)
end
```

- update the recipes show page to show the image

```
<p>
	<%= image_tag @recipe.image %>
</p>
```

### Adding cocoon gem

- add the cocoon gem
- bundle
- update the app.js

```
//= require jquery
//= require rails-ujs
//= require bootstrap-sprockets
//= require cocoon
//= require activestorage
//= require turbolinks
//= require_tree .

```

- rails g model Ingredient name recipe:belongs_to
- rails g model Direction step:text recipe:belongs_to
- rails db:migrate
- update recipe.rb with the ingredient and direction
- inverse of is [explained here](https://www.rubydoc.info/gems/cocoon/1.2.12)

```
	has_many :ingredients, inverse_of: :recipe
	has_many :directions, inverse_of: :recipe

	accepts_nested_attributes_for :ingredients,
  															reject_if: proc { |attributes| attributes['name'].blank? },
  															allow_destroy: true
 	accepts_nested_attributes_for :directions,
  															reject_if: proc { |attributes| attributes['step'].blank? },
  															allow_destroy: true

  validates :title, :description, :image, presence: true
```

- update the params in the recipes controller

```
def recipe_params
  params.require(:recipe).permit(:title, :description, :user_id, :image,
                                  ingredients_attributes: [:id, :name, :_destroy], 
                                  directions_attributes: [:id, :step, :_destroy])
end
```

- create the partial recipes/ingredient_fields.html.haml

```
.form-inline.clearfix
	.nested-fields
		= f.input :name, input_html: { class: 'form-input form-control' }
		= link_to_remove_association "Remove", f, class: "form-button btn btn-default"
```

- create the partial recipes/direction_fields.html.haml

```
.form-inline.clearfix
	.nested-fields
		= f.input :step, input_html: { class: 'form-input form-control' }
		= link_to_remove_association "Remove Step", f, class: "btn btn-default form-button"
```

- add devise gem
- rails g devise:install
- rails g devise:views
- rails g devise User
- rails db:migrate
- update user.rb

```
has_many :recipes
```

- update recipe.rb

```
belongs_to :user
```

- 